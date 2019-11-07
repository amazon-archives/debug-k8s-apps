+++
weight=1
+++

## EKS architecture
- AWS Managed Control Plane
- Highly available
- AWS IAM authentication
- VPC networking

---

# etcd design 
- Minimum 3 etcd servers 
- Spread across availability zones

---

# Network considerations
- EKS cluster endpoint can be public or private
- EKS uses aws-vpc-cni
- Worker nodes and Pods get VPC IP

---

# [aws-vpc-cni](https://github.com/aws/amazon-vpc-cni-k8s)
- Pods recieve an IP address from a VPC subnet
- No IP = Pod Pending
- Plan for growth
- [customize cni variables](https://docs.aws.amazon.com/eks/latest/userguide/cni-env-vars.html)

---
# [use CNI Metrics Helper](https://docs.aws.amazon.com/eks/latest/userguide/cni-metrics-helper.html)
![](https://docs.aws.amazon.com/eks/latest/userguide/images/EKS_CNI_metrics.png)

