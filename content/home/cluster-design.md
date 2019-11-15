+++
weight = 10
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

### eksctl configuration file

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: debug-k8s
  region: us-east-1

nodeGroups:
  - name: nodegroup
    instanceType: m5.xlarge
    desiredCapacity: 4
    ssh:
      allow: true
      publicKeyName: arun-us-east1

cloudWatch:
  clusterLogging:
    # enable specific types of cluster control plane logs
    enableTypes: ["audit", "authenticator", "controllerManager"]
    # all supported types: "api", "audit", "authenticator", "controllerManager", "scheduler"
    # supported special values: "*" and "all"
```

