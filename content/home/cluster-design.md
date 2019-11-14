+++
weight=1
+++

{{% section %}}

### Kubernetes Components

![](images/k8s-cluster-1.png)

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


<!-- 
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

-->

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

### Coredns scaling

> coreDNSMemory required in MB = (Pods + Services)/1000 + 54

- Scale CoreDNS pods 
```
kubectl -n kube-system scale --current-replicas=2
 --replicas=10 deployment/coredns
```
- Node-local DNS [addon](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/nodelocaldns)
    - CoreDNS DaemonSet on each node


{{% /section %}}
