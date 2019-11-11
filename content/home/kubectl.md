+++
weight = 2
+++

<section data-noprocess>
 <h2>EKS Architecture</h2>
 <img src="https://d1.awsstatic.com/Getting%20Started/eks-project/EKS-demo-app.e7ce7b188f2662b8573b5881a6b843e09caf729a.png" height=400>
</section>

---

### How does kubectl work

{{% fragment %}} - kubectl communicates with the Kubernetes API server {{% /fragment %}}

{{% fragment %}} - uses a configuration file generally located at ~/.kube/config {{% /fragment %}}

{{% fragment %}} - In EKS, kubectl + aws-iam-authenticator = ❤️ {{% /fragment %}}

---

### If your kubectl cannnot connect to your Kubernetes cluster

{{% fragment %}}1. Update [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) & [aws-iam-authenticator](https://kubernetes.io/docs/tasks/tools/install-kubectl/) {{% /fragment %}}

{{% fragment %}}2. Update kubeconfig using AWS cli  {{% /fragment %}}


{{% fragment %}}3. Is cluster endpoint accessible?  {{% /fragment %}}

---

### Check if cluster is accessible

``` 
curl -k http://CLUSTER_ENDPOINT/api/v1
```

response:

```
"kind": "APIResourceList",
  "groupVersion": "v1",
  "resources": [
  {
        "name": "bindings",
              "singularName": "",
                    "namespaced": true,
                          "kind": "Binding",
                          "verbs": [
                                  "create"
...                                        
```

---

#### Use AWS CLI to auto-generate kube config file
```
aws eks update-kubeconfig --name {cluster-name} 
```

---
*cat ~/.kube/config*

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: {REDACTED} 
    server: https://DFEA886AB17A069545SJDS9F06BCE3DCC.gr7.us-west-2.eks.amazonaws.com
  name: arn:aws:eks:us-west-2:09123456789:cluster/eks1
contexts:
- context:
    cluster: arn:aws:eks:us-west-2:09123456789:cluster/eks1
    user: arn:aws:eks:us-west-2:09123456789:cluster/eks1
  name: arn:aws:eks:us-west-2:09123456789:cluster/eks1
current-context: arn:aws:eks:us-west-2:09123456789:cluster/eks1
kind: Config
preferences: {}
users:
	- name: arn:aws:eks:us-west-2:09123456789:cluster/eks1
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - token
      - -i
      - eks1
      command: aws-iam-authenticator
```
---
### aws-auth config map
```  
kubectl -n kube-system describe configmap aws-auth
```

```
Name:         aws-auth
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
mapRoles:
----
- groups:
  - system:bootstrappers
  - system:nodes
  rolearn: arn:aws:iam::09123456789:role/eksctl-eks-nodegroup-ng
  username: system:node:{{EC2PrivateDNSName}}

mapUsers:
----
- userarn: arn:aws:iam::09123456789:user/realvarez
  groups:
    - system:masters

```


---

### kubectl works!
```
kubectl cluster-info
```
output:
```
Kubernetes master is running at https://xxxx.y.region.eks.amazonaws.com
```
