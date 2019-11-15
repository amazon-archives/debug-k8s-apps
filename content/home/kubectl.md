+++
weight = 40
+++

<section data-noprocess>
 <h2>EKS Architecture</h2>
 <img src="https://d1.awsstatic.com/Getting%20Started/eks-project/EKS-demo-app.e7ce7b188f2662b8573b5881a6b843e09caf729a.png" height=400>
</section>

---

<section data-markdown>
### How does kubectl work
  <script>
### How does kubectl work

- kubectl communicates with the Kubernetes API server <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="1" --> 
- uses a configuration file generally located at ~/.kube/config <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="2" -->
- In EKS, kubectl + aws-iam-authenticator = ❤️ <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="3" -->
  </script>
</section>

---

### Kubectl on client-side

{{% fragment %}}- Validation {{% /fragment %}}

{{% fragment %}}- Infer generators, explicitly specified using `--generator` {{% /fragment %}}

{{% fragment %}}- Create a Runtime object using generators {{% /fragment %}}

{{% fragment %}}- API version negotiation {{% /fragment %}}

{{% fragment %}}- REST request created {{% /fragment %}}

{{% fragment %}}- Authn {{% /fragment %}}

---

### Kubectl on server-side

{{% fragment %}}- Authn and authz {{% /fragment %}}

{{% fragment %}}- Admission controllers {{% /fragment %}}

{{% fragment %}}- Deserializes HTTP request to etcd {{% /fragment %}}

{{% fragment %}}- Initializers {{% /fragment %}}

{{% fragment %}}- Deployment, ReplicaSet, Scheduler controller {{% /fragment %}}

{{% fragment %}}- Kubelet queries API server (every 20 secs) for pods {{% /fragment %}}

{{% fragment %}}- Identify CRI and "pause" containers {{% /fragment %}}

{{% fragment %}}- CNI plugin (IP address allocation) {{% /fragment %}}

{{% fragment %}}- Container startup (pull image, start using CRI) {{% /fragment %}}

{{% note %}}
https://github.com/jamiehannaford/what-happens-when-k8s

- kube-apiserver exposes its schema document (in OpenAPI format) at this path, it's easy for clients to perform their own discovery. To improve performance, kubectl also caches the OpenAPI schema to the ~/.kube/cache/discovery directory.

- "pause" container which hosts all of the namespaces to allow inter-pod communication.
{{% /note %}}


---

### ... have you ever ran into this?
```
$ kubectl get svc 
error: the server doesn't have a resource type "svc"
```

{{% fragment %}}1. Update [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) & [aws-iam-authenticator](https://kubernetes.io/docs/tasks/tools/install-kubectl/) {{% /fragment %}}

{{% fragment %}}2. Update kubeconfig using AWS cli  {{% /fragment %}}


{{% fragment %}}3. Is cluster endpoint accessible?  {{% /fragment %}}

---

### Check if cluster is accessible

``` 
$ curl -k http://CLUSTER_ENDPOINT/api/v1
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
$ aws eks update-kubeconfig --name {cluster-name} 
```

---
```
$ cat ~/.kube/config
-----
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
$ kubectl -n kube-system describe configmap aws-auth
-----
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

If you are not part of the aws-auth configmap, then you'll see this,
```
error: You must be logged in to the server (Unauthorized)
```

get your arn added to the aws-auth configmap

```
mapUsers:
 ----
  - userarn: arn:aws:iam::09123456789:user/{YOUR_USER_ARN_HERE}
  -    groups:
  -         - system:masters
  -          
```

---

### kubectl works!

```
$ kubectl cluster-info
-----
Kubernetes master is running at https://xxxx.y.region.eks.amazonaws.com
```

check cluster status:

``` 
$ kubectl get componentstatus 
-----
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   

```
