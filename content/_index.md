+++
title = "Tutorial: Debug Your Kubernetes Apps - Arun Gupta & Re Alvarez Parmar, Amazon"
outputs = ["Reveal"]
[favicon]
src = "images/favicon.ico"
[logo]
src = "images/aws-logo.svg"
alt = "AWS Smile!"
[reveal_hugo]
margin = 0.2
slide_number = true
controls = false
custom_theme = "reveal-hugo/themes/robot-lung.css"
+++

# Tutorial: Debug Your Kubernetes Apps
 Arun Gupta & Re Alvarez Parmar, Amazon


---

In this presentation we will learn how to troubleshoot Kubernetes applications. 

---

Agenda slide

---

![](images/eks-arch.jpg)

### EKS Architecture

---

## Kubernetes Components


- Master node

- Worker Node

- kubectl (User)

---

# How to setup a Kubernetes cluster

{{% fragment %}} 1. Create Master Nodes (aka the Control Plane) {{% /fragment %}}

{{% fragment %}} 2. use *kubectl* to connect to the Control Plane {{% /fragment %}}

{{% fragment %}} 3. Install Worker nodes {{% /fragment %}}

{{% fragment %}} 4. Deploy apps and add-ons {{% /fragment %}}

{{% fragment %}} 5. ... profit? {{% /fragment %}}

--- 

You create your control plane and fire up kubectl
{{% fragment %}} and then this happens... {{% /fragment %}}

{{% fragment %}} ![](images/kubectl-fail.png) or `The connection to the server localhost:8000 was refused` {{% /fragment %}}

---

Things that go wrong in kubectl configuration

{{% fragment %}} 1. Does kubeconfig exist? Check kubeconfig in ~/.kube/ directory {{% /fragment %}}
{{% fragment %}} 2. In case of EKS, is aws-iam-authenticator installed? {{% /fragment %}}
