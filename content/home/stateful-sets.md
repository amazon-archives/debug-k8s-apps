+++
weight = 4
+++

# What about stateful containers?

---

### Pods are ephemeral by nature 

- Volumes allow pods to persist data

- Volumes are accessible to all containers in a pod

- Data can persist even after pod termination

---

### PersistentVolume (PV) 

> pre-provisioned storage in the cluster or dynamically provisioned by storage class


### PersistentVolumeClaim (PVC)

> request for storage by a user

### StorageClass

> administrator provided "classes" of storage

{{% note %}}
Persistent Volumes provide a plugin model for storage in Kubernetes. How storage is provided is abstracted from how it is consumed

{{% /note %}}

---

### What are statefulsets?
- Provide pods with storage to persist data
- Create PVC dynamically using `volumeClaimTemplates`
- Each pod gets its own dedicated PVC
- Create headless service type (`clusterIP: None`) 
- Ordered deployment and scaling
{{% note %}}
“As you probably realize, we said PVCs are created and associated with the Pods, but we didn’t say anything about PVs. That is because StatefulSets do not manage PVs in any way. The storage for the Pods must be provisioned in advance by an admin, or provisioned on-demand by a PV provisioner based on the requested storage class and ready for consumption by the stateful Pods.”
{{% note %}}

---

### Step 1. Create storage class
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: mysql-gp2
  provisioner: kubernetes.io/aws-ebs
  parameters:
    type: gp2
    reclaimPolicy: Delete
    mountOptions:
    - debug
```

---

### Step 2. use volumeClaimTemplates in the Statefulset

```
volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: mysql-gp2
      resources:
        requests:
          storage: 10Gi
```

---
### mysql statefulsets on Kubernetes

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  spec:
    selector:
      matchLabels:
        app: mysql
  serviceName: mysql
  replicas: 3
  template:
    metadata:
      labels:
        app: mysql
    spec:
      initContainers:
   ...

        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
          subPath: mysql

  ...
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: mysql-gp2
      resources:
        requests:
          storage: 10Gi
```
See full yaml [here](https://eksworkshop.com/statefulset/statefulset.files/mysql-statefulset.yml)

---

### Persistent storage options in EKS
- [EBS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
- [EFS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html)
- [FSx for Lustre CSI Driver (Alpha)](https://github.com/kubernetes-sigs/aws-fsx-csi-driver)

---
### EKS + EBS
- Persistent volumes can be [dynamically](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/dynamic-provisioning) or 
[statically](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/static-provisioning) provisioned
- Volumes can be [resized](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/resizing) and support [Snapshots](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/snapshot)
- Volumes are local to AZ, use EFS where possible

