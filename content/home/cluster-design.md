+++
weight=1
+++

### EKS architecture
- AWS Managed Control Plane
- Highly available
- AWS IAM authentication
- VPC networking

---

### etcd design 
- Minimum 3 etcd servers 
- Spread across availability zones

---

### Network considerations
- EKS cluster endpoint can be public or private
- EKS uses aws-vpc-cni
- Worker nodes and Pods get VPC IP

---

### [aws-vpc-cni](https://github.com/aws/amazon-vpc-cni-k8s)
- Pods recieve an IP address from a VPC subnet
- Max number of pods is limited by EC2 Instance size
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

### Core-dns scaling
> Memory required in MB = (Pods + Services)/1000 + 54

- Scale CoreDNS pods 
```
kubectl -n kube-system scale --current-replicas=2
 --replicas=10 deployment/coredns
```
- Node-local DNS [addon](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/nodelocaldns)
    - CoreDNS DaemonSet on each node
