+++
weight = 1
+++


**How does kubectl work**

{{% fragment %}} - kubectl communicates with the Kubernetes API server {{% /fragment %}}

{{% fragment %}} - uses a configuration file generally located at ~/.kube/config {{% /fragment %}}

{{% fragment %}} - In EKS, kubectl + aws-iam-authenticator = ❤️ {{% /fragment %}}

---
**If your kubectl cannnot connect to your Kubernetes cluster**

{{% fragment %}}1. Update kubectl & aws-iam-authenticator {{% /fragment %}}

{{% fragment %}}2. Update kubeconfig using AWS cli  {{% /fragment %}}
{{% fragment %}} ```aws eks update-kubeconfig --name {cluster-name}``` {{% /fragment %}}

