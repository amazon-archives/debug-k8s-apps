+++
weight = 20
+++

{{% section %}}

## Section 2:
## Networking

---

### Networking

- Container Networking Interface (CNI)
- CoreDNS (DNS Server)

---

## Section 2.1:
## Networking
## Container Networking Interface (CNI)

---


### Container Networking Interface

[Container Networking Interface](https://github.com/containernetworking/cni) consists of a [specification](https://github.com/containernetworking/cni/blob/master/SPEC.md), libraries for writing plugins to configure network interface in Linux containers, and a number of supported plugins.

- EKS uses [amazon-vpc-cni](https://github.com/aws/amazon-vpc-cni-k8s)
- Worker nodes and pods get IP address from VPC


---

### Amazon VPC CNI for Kubernetes

- Elastic Network Interface (ENI) attached to the worker node
- Pods recieve an IP address from a VPC subnet
- Max number of pods is limited by ENIs that can be attached to EC2 instance
  - defined at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html
- No IP = Pod Pending
- Plan for growth
- [Customize cni variables](https://docs.aws.amazon.com/eks/latest/userguide/cni-env-vars.html)

---

Create a deployment:

```
$ kubectl create -f resources/manifests/hello-deployment.yaml 
```

---

Scale to 240 replicas

```
$ kubectl scale --replicas=240 deployment hello
```

---

Get deployments:

```
$ kubectl get deployments
NAME    READY     UP-TO-DATE   AVAILABLE   AGE
hello   222/240   240          222         5m27s
```

---

Get pending pods:

```
$ kubectl get pods --field-selector=status.phase==Pending
NAME                    READY   STATUS    RESTARTS   AGE
hello-9fdd9558f-2x8d8   0/1     Pending   0          3m55s
hello-9fdd9558f-4s4hl   0/1     Pending   0          3m55s
hello-9fdd9558f-5fsfv   0/1     Pending   0          3m55s
hello-9fdd9558f-5jffb   0/1     Pending   0          3m55s
hello-9fdd9558f-69f6x   0/1     Pending   0          3m54s
hello-9fdd9558f-6pfzb   0/1     Pending   0          3m55s
hello-9fdd9558f-7l844   0/1     Pending   0          3m55s
hello-9fdd9558f-8zmhk   0/1     Pending   0          3m55s
hello-9fdd9558f-d48ng   0/1     Pending   0          3m55s
hello-9fdd9558f-hqpwp   0/1     Pending   0          3m55s
hello-9fdd9558f-jjs7b   0/1     Pending   0          3m55s
hello-9fdd9558f-jsqsv   0/1     Pending   0          3m55s
hello-9fdd9558f-kjjt4   0/1     Pending   0          3m55s
hello-9fdd9558f-m7fnc   0/1     Pending   0          3m55s
hello-9fdd9558f-pp9ls   0/1     Pending   0          3m55s
hello-9fdd9558f-sqnvg   0/1     Pending   0          3m55s
hello-9fdd9558f-v4m4z   0/1     Pending   0          3m54s
hello-9fdd9558f-vcsx6   0/1     Pending   0          3m55s
```

---

Get events:

```
$ kubectl get events --field-selector involvedObject.kind=Pod,type=Warning
```

---

Shows the output:

```
11m         Warning   FailedCreatePodSandBox   pod/hello-9fdd9558f-zns6z   Failed create pod sandbox: rpc error: 
                                                                           code = Unknown desc = failed to set 
                                                                           up sandbox container "2f43374edb1fdc76
                                                                           74587fab2f040bdc8e29abb4cf0ac7a12daea4
                                                                           f04bab8fe4" network for pod 
                                                                           "hello-9fdd9558f-zns6z": 
                                                                           NetworkPlugin cni failed to set up 
                                                                           pod "hello-9fdd9558f-zns6z_default" 
                                                                           network: add cmd: failed to assign an 
                                                                           IP address to container
```

---

### IP address allocation

ipamD (IP Address Management Daemon):

- maintain a warm-pool of available IP addresses

- assign an IP address to a Pod

Each ENI uses 1 IP address to attach to the instance.

```
Maximum number of IPs = min((number of ENIs * (IP addresses per ENI - 1)), free IPs in the subnet)
```

---

### IP Addresses per Node

For a **m5.xlarge** (4 ENIs, 15 IP addresses/ENI) node type:

{{% fragment %}}Maximum number of IP addresses per host by ENIs is **56 = 4 * (15 - 1)**{{% /fragment %}}

{{% fragment %}}Default subnet is **192.168.0.0/19** => 8192 IPs{{% /fragment %}}

{{% fragment %}}Maximum number of IP addresses per host is **min(56, 8192)**{{% /fragment %}}

{{% fragment %}}**v1.16** [recommends](https://kubernetes.io/docs/setup/best-practices/cluster-large/) no more than 100 pods per node{{% /fragment %}}

---

### CNI Metrics Helper

[CNI Metrics Helper](https://docs.aws.amazon.com/eks/latest/userguide/cni-metrics-helper.html) helps you track how many IP addresses have been assigned and how many are available.

The following metrics are collected for your cluster and exported to CloudWatch:

- Maximum number of ENIs that the cluster can support
- Number of ENIs have been allocated to pods
- Number of IP addresses currently assigned to pods
- Total and maximum numbers of IP addresses available
- Number of ipamD errors

---

### Create CNI Metrics Helper Policy

```
aws iam create-policy \
  --policy-name CNIMetricsHelperPolicy \
  --description "Grants permission to write metrics to CloudWatch" \
  --policy-document file://./resources/manifests/cni-metrics-policy.json
```

---

Attach policy to the worker nodes IAM role:

```
$ ROLE_NAME=$(aws iam list-roles \
  --query \
  'Roles[?contains(RoleName,`debug-k8s-nodegroup`)].RoleName' --output text)
aws iam attach-role-policy \
  --role-name $ROLE_NAME \
  --policy-arn arn:aws:iam::091144949931:policy/CNIMetricsHelperPolicy
```

---

### Deploy CNI metrics helper

```
$ kubectl apply -f \
    https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/release-1.5/config/v1.5/cni-metrics-helper.yaml
...
$ kubectl get deployment cni-metrics-helper -n kube-system
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
cni-metrics-helper   1/1     1            1           60s
```

---

### CNI Metrics Helper for Amazon EKS Cluster


<img src="images/cni-metrics-helper.png"/>

<!--
![](https://docs.aws.amazon.com/eks/latest/userguide/images/EKS_CNI_metrics.png)
-->

---

## Section 2.2:
## Networking
## CoreDNS (DNS Server)

---

### CoreDNS

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

Add **log** in Corefile

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

### Scaling CoreDNS

- Node-local DNS [addon](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/nodelocaldns)
    - CoreDNS DaemonSet on each node
- Scale CoreDNS pods 
```
$ kubectl -n kube-system scale --replicas=4 deployment/coredns
```

---

### Sizing CoreDNS

- [Memory required in MB](https://github.com/coredns/deployment/blob/master/kubernetes/Scaling_CoreDNS.md)

> (Pods + Services)/1000 + 54

{{% note %}}Max memory: 19MB, Operating buffer: 5MB, Cache: 30MB{{% /note %}}

- Amazon EKS configuration:
```
Limits:
  memory:  170Mi
Requests:
  cpu:        100m
  memory:     70Mi
```

{{% note %}}
In large scale Kubernetes clusters, CoreDNS’s memory usage is predominantly affected by the number of Pods and Services in the cluster.
{{% /note %}}


---

### CoreDNS *autopath* plugin

- Optional CoreDNS plugin
- Improves performance for queries of names external to the cluster
- Requires CoreDNS to use more memory
- Memory with **autopath** plugin enabled

> (Pods + Services)/250 + 56


{{% /section %}}
