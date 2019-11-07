+++
title = "Tutorial: Debug Your Kubernetes Apps - Arun Gupta & Re Alvarez Parmar, Amazon"
outputs = ["Reveal"]
[favicon]
src = "images/favicon.ico"
[logo]
src = "images/aws-logo.svg"
alt = "AWS Smile!"
[reveal_hugo]
theme = "white"
margin = 0.1
width = "80%"
height = "80%"
slide_number = true
controls = false
progress = false
hash = false
transition = 'fade'
+++

<style type="text/css">
  .reveal {
    font-size: 30px;
  }
  .reveal p {
    text-align: left;
    font-size: 30x;
  }
  .reveal ul {
    display: block;
    font-size: 30px;
  }
  .reveal ol {
    display: block;
    font-size: 30px;
  }
  .reveal code {
   font-size: 15px;
  } 
  .reveal pre code {
   font-size: 15px;
  }
</style>

# Tutorial: Debug Your Kubernetes Apps
### Arun Gupta & Re Alvarez Parmar, Amazon


---

In this presentation we will learn how to troubleshoot Kubernetes applications. 

---

Agenda slide

---

![](images/eks-arch.jpg)

**EKS Architecture**

---

**Kubernetes Components**


- Master node

- Worker Node

- kubectl (User)

---

**How to setup a Kubernetes cluster**

{{% fragment %}} 1. Create Master Nodes (aka the Control Plane) {{% /fragment %}}

{{% fragment %}} 2. use *kubectl* to connect to the Control Plane {{% /fragment %}}

{{% fragment %}} 3. Install Worker nodes {{% /fragment %}}

{{% fragment %}} 4. Deploy apps and add-ons {{% /fragment %}}

{{% fragment %}} 5. ... profit? {{% /fragment %}}

