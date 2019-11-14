+++
weight=1
+++


### Kubernetes Components

<img src="images/k8s-cluster-1.png" height="75%" width="75%"/>


---

### How to setup a Kubernetes cluster

{{% fragment %}} 1. Create Controller nodes (aka the Control Plane) {{% /fragment %}}

{{% fragment %}} 2. Setup *etcd* and connect to Controller {{% /fragment %}}

{{% fragment %}} 3. use *kubectl* to connect to the Control Plane {{% /fragment %}}

{{% fragment %}} 4. Install Worker nodes {{% /fragment %}}

{{% fragment %}} 5. Deploy apps and add-ons {{% /fragment %}}

{{% fragment %}} 6. ... profit? {{% /fragment %}}

---

### Design considerations for k8s cluster

- Control Plane
- Data Plane
- What EC2 instance types?
- What operating system?
- How many pods per cluster?
- Etcd co-located with master?
- Secure and backup etcd, how often?
- High availability
- Disaster recovery
- Upgrade k8s versions
- Staying up to date with k8s release cadence
- Patching application and k8s cluster for CVEs and security patches
- Monitoring and logging
- One big cluster, multiple small clusters
- Blast radius


---

### Amazon EKS Architecture

![](images/k8s-cluster-2.png){ width=80%, height=80% } 

---

### EKS core tenets

- Platform for enterprises to run production grade workloads
- Provide a native and upstream experience (CNCF Certified)
- Provide seamless integration with AWS services
- Actively contribute to upstream project

---

### Kubectl

One CLI to control your k8s cluster

![](images/k8s-cluster-3.png)

---

### EKS architecture

- AWS Managed Control Plane
  - Master nodes
  - etcd cluster nodes
  - NLB for API load-balancing
- Highly available
- AWS IAM authentication
- VPC networking


---

### Master node components

- **apiserver:** exposes APIs for  master nodes 
- **scheduler:** decides which pod should run on which worker node
- **controller manager:** makes changes attempting to move the current state towards the desired state
- **etcd:** key/value data store used to store cluster state

---

### etcd design 
- Minimum 3 etcd servers 
- Spread across availability zones
- Uses RAFT protocol

---

### Worker node components
- [**kubelet:**](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) handles communication between worker and master nodes
- [**kube-proxy:**](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) handles communication between pods, nodes, and the outside world
- **container runtime:** runs containers on the node.


---

### Create EKS cluster

```
eksctl create cluster -f resources/manifests/eks-cluster.yaml
```

---

### Network considerations

- EKS cluster endpoint can be public or private
- EKS uses amazon-vpc-cni
- Worker nodes and Pods get VPC IP

---

### [aws-vpc-cni](https://github.com/aws/amazon-vpc-cni-k8s)

- Pods recieve an IP address from a VPC subnet
- Max number of pods is limited by EC2 Instance size
  - `m5.xlarge` - 4 ENIs, 15 IPv4 addresses per interface
  - defined at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html
- No IP = Pod Pending
- Plan for growth
- [customize cni variables](https://docs.aws.amazon.com/eks/latest/userguide/cni-env-vars.html)


---

# [use CNI Metrics Helper](https://docs.aws.amazon.com/eks/latest/userguide/cni-metrics-helper.html)
![](https://docs.aws.amazon.com/eks/latest/userguide/images/EKS_CNI_metrics.png)


---

### Kubelet resource reservation

- Monitor kubelet on the worker node

```
journalctl -u kubelet
```

- Use kube-reserved to reserve resources for kubelet, container runtime & node problem detector

```
--kube-reserved=[cpu-100m][,][memory=100Mi][,]
[ephemeral-storage=1Gi][,][pid=1000]
```

- Use system-reserved to reserve resources for system daemons liks sshd, udev, kernel
```
--system-reserved=[cpu-100m][,][memory=100Mi][,]
[ephemeral-storage=1Gi][,][pid=1000]
```

---

### Kubelet resource reservation for Amazon EKS using User Data

```
ADD EXAMPLE
```

---

### Kubelet resource reservation for Amazon EKS using eksctl

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: myeks
  region: us-east-1

nodeGroups:
  - name: myng
    instanceType: m5.xlarge
    desiredCapacity: 4
    kubeletExtraConfig:
        kubeReserved:
            cpu: "300m"
            memory: "300Mi"
            ephemeral-storage: "1Gi"
        kubeReservedCgroup: "/kube-reserved"
        systemReserved:
            cpu: "300m"
            memory: "300Mi"
            ephemeral-storage: "1Gi"
        evictionHard:
            memory.available:  "200Mi"
            nodefs.available: "10%"
        featureGates:
            DynamicKubeletConfig: true
            RotateKubeletServerCertificate: true # has to be enabled, otherwise it will be disabled
```

---

### Avoid oversubscription
- Use resource limits and requests

    ```
    Pod.sepc.containers.resources: 
                          limits: 
                            memory: 128Mi 
                            cpu: 500m 
                          requests:
                            memory: 64Mi 
                            cpu: 250m
    ```
- Use resource quotas on namespaces
    ```
    apiVersion: v1 
    kind: ResourceQuota 
    metadata: 
      name: resource-quota 
        namespace: quota-ns
    spec: 
      hard: 
          limits.cpu: 2
    ```

---

Once the quota on the namespace is full, newer pods will fail

```
$ kubectl  get events
...
Error creating: pods "nginx-fb556779d-xmsgv" is forbidden: 
exceeded quota: resource-quota, requested: limits.cpu=500m, used: limits.cpu=2, limited: limits.cpu=2
```

---
Use LimitRange to set default limit

```
apiVersion: v1 
kind: LimitRange 
metadata: 
  name: limit-range-example 
    namespace: limit-range-example 
    spec: 
      limits: 
        - default: 
            memory: 512Mi 
            cpu: 1 
          defaultRequest: 
            memory: 256Mi 
            cpu: 500m 
          type: Container
```

{{% note %}}
“When you are using quotas on a namespace, one requirement is that every container in the namespace must have resource limits and requests defined. Sometimes this requirement can cause complexity and make it more difficult to work quickly with Kubernetes. Specifying resource limits correctly, while an essential part of preparing an application for production, does add additional overhead when, for example, using Kubernetes as a platform for development or testing workloads.”
{{% note %}}

---

```
$ kubectl describe limitranges 
Name:       limit-mem-cpu-per-container
Namespace:  limitrange
Type        Resource  Min   Max   Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---   ---------------  -------------  -----------------------
Container   cpu       100m  800m  110m             700m           -
Container   memory    99Mi  1Gi   111Mi            900Mi          -
```
---

### Limits scenarios 
(container limits and requests override Limitranges)

1. If container defines both *limits* and *requests*, *LimitRanges* will have no effect
2. If container has no *limits* or *requests*, *LimitRanges* will be inherited
3. If container has *requests* but not *limits*, *limits* will be inherited from *LimitRanges*
4. If container has *limits* defined, then *requests* = *limits*, *LimitRanges* will have no effect

---

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

