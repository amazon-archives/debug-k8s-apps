+++
weight=2
+++


### Network considerations

- EKS cluster endpoint can be public or private
- EKS uses amazon-vpc-cni
- Worker nodes and Pods get VPC IP

---

### [amazon-vpc-cni](https://github.com/aws/amazon-vpc-cni-k8s)

- Pods recieve an IP address from a VPC subnet
- Max number of pods is limited by EC2 instance size
  - `m5.xlarge` - 4 ENIs, 15 IPv4 addresses per interface
  - defined at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html
- No IP = Pod Pending
- Plan for growth
- [customize cni variables](https://docs.aws.amazon.com/eks/latest/userguide/cni-env-vars.html)


### Troubleshooting coreDNS

* CoreDNS was GA in 1.11
* CoreDNS uses [Corefile](https://coredns.io/2017/07/23/corefile-explained/) for configuration
* Corefile can be edited by editing coredns configmap

---

### Check CoreDNS pods are running

```
$ kubectl get pods --namespace=kube-system -l k8s-app=kube-dns
---
NAME                       READY   STATUS    RESTARTS   AGE
coredns-79d667b89f-hcwnr   1/1     Running   0          63d
coredns-79d667b89f-qh8cd   1/1     Running   0          63d
```



### Check if CoreDNS service is up

```
$ kubectl get svc --namespace=kube-system
---
kubectl get svc kube-dns --namespace=kube-system             
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   10.100.0.10   <none>        53/UDP,53/TCP   63d
```
---

## Enable logging in CoreDNS
add **log** in Corefile

```
$ kubectl -n kube-system edit configmap coredns
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
    namespace: kube-system
    data:
      Corefile: |
      .:53 {
          log
          errors
          health
          kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            upstream
            fallthrough in-addr.arpa ip6.arpa
            } 
          prometheus :9153
          proxy . /etc/resolv.conf
          cache 30
          loop
          reload
          loadbalance
      }
```

--- 

### Check CoreDNS logs

```
$ for p in $(kubectl get pods --namespace=kube-system -l k8s-app=kube-dns -o name); do kubectl logs --namespace=kube-system $p; done

.:53
2019-09-12T16:14:47.907Z [INFO] CoreDNS-1.2.6
2019-09-12T16:14:47.907Z [INFO] linux/amd64, go1.10.8, 756749c5
CoreDNS-1.2.6
linux/amd64, go1.10.8, 756749c5
 [INFO] plugin/reload: Running configuration MD5 = 2e2180a5eeb3ebf92a5100ab081a6381
 W1002 06:56:56.326426       1 reflector.go:341] github.com/coredns/coredns/plugin/kubernetes/controller.go:318: watch of *v1.Namespace ended with: too old resource version: 123473 (4874440)
 W1002 22:44:07.890987       1 reflector.go:341] github.com/coredns/coredns/plugin/kubernetes/controller.go:318: watch of *v1.Namespace ended with: too old resource version: 4874440 (5041806)
 W1002 22:44:07.893920       1 reflector.go:341] github.com/coredns/coredns/plugin/kubernetes/controller.go:311: watch of *v1.Service ended with: too old resource version: 123345 (5041806)
 W1008 00:10:53.440168       1 reflector.go:341] github.com/coredns/coredns/plugin/kubernetes/controller.go:311: watch of *v1.Service ended with: too old resource version: 5041806 (6327012)
```

---

### Coredns scaling

> coreDNSMemory required in MB = (Pods + Services)/1000 + 54

- Scale CoreDNS pods 
```
kubectl -n kube-system scale --current-replicas=2
 --replicas=10 deployment/coredns
```
- Node-local DNS [addon](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/nodelocaldns)
    - CoreDNS DaemonSet on each node

---

### CoreDNS *autopath* plugin
- Optional CoreDNS plugin
- Improves performance for queries of names external to the cluster
- Requires CoreDNS to use more memory


> coreDNS Memory with **autopath** required in MB =
>
>(Pods + Services)/250 + 56

