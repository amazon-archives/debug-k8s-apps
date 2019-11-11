+++
weight = 4
+++

# What about stateful containers?

---

### Persistent storage in EKS
- [EBS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
- [EFS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html)
- [FSx for Lustre CSI Driver (Alpha)](https://github.com/kubernetes-sigs/aws-fsx-csi-driver)

---
### EKS + EBS
- Persistent volumes can be [dynamically](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/dynamic-provisioning) or 
[statically](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/static-provisioning) provisioned
- Volumes can be [resized](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/resizing) and support [Snapshots](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/snapshot)
- Volumes are local to AZ, use EFS where possible

