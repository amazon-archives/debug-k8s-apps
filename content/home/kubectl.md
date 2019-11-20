+++
weight = 30
+++

{{% section %}}

## Section 3:
## Using Kubectl

---

#### EKS Architecture
<img src="https://d1.awsstatic.com/Getting%20Started/eks-project/EKS-demo-app.e7ce7b188f2662b8573b5881a6b843e09caf729a.png" height="500" />

---


### How does kubectl work
{{< frag c="- kubectl communicates with the Kubernetes API server" >}}

{{< frag c="- uses a configuration file generally located at ~/.kube/config" >}}

{{< frag c="- In EKS, kubectl + aws-iam-authenticator = ❤️" >}}

---

### Kubectl on client-side

**Validation**

- Infer generators, explicitly specified using *--generator*

- Create a Runtime object using generators

- API version negotiation

- REST request created

- Authn

---

### Kubectl on server-side

- Authn and authz 

- Admission controllers

- Deserializes HTTP request to etcd 

- Initializers

- Deployment, ReplicaSet, Scheduler controller

- Kubelet queries API server (every 20 secs) for pods

- Identify CRI and "pause" containers

- CNI plugin (IP address allocation)

- Container startup (pull image, start using CRI)

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

1. Update [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) & [aws-iam-authenticator](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

2. Update kubeconfig using AWS cli or eksctl

3. Is cluster endpoint accessible?

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

{{% /section %}}
