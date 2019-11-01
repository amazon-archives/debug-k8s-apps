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

## How to setup a Kubernetes cluster

1. Create Master Nodes (aka the Control Plane)

2. use *kubectl* to connect to the Control Plane

3. Install Worker nodes

4. Deploy apps and add-ons

5. ... profit? 

--- 

You create your control plane and fire up kubectl and then this happens...

![](images/kubectl-fail.png)

---

## Things that go wrong in kubectl 

1. Does kubeconfig file exist?

   check kubeconfig in ~/.kube/ directory 
