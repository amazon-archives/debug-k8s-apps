+++
weight=1
+++

## EKS architecture
- AWS Managed Control Plane
- Highly available
- Managed etcd 
- AWS IAM authentication

---

# Self-managed Kubernetes architecture

---

# ETCd design
- Minimum 3 etcd servers 
- Spread across availability zones

---

# Network considerations

---

# EKS architecture - Nodes 

---

# aws-vpc-cni
- Pods recieve an IP address from a VPC subnet
- No IP = Pod Pending

---

# cni customization
