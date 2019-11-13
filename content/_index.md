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
    font-size: 25px;
  }
  .reveal h3 {
    text-align: left;
  }
  .reveal ul {
    display: block;
    font-size: 25px;
  }
  .reveal ol {
    display: block;
    font-size: 25px;
  }
  .reveal code {
   font-size: 15px;
  } 
  .reveal pre code {
   font-size: 15px;
  }
  .reveal section img {
  border-style: none;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
  }
</style>

# Tutorial: Debug Your Kubernetes Apps
#### Arun Gupta & Re Alvarez Parmar, Amazon


---

In this presentation we will learn how to troubleshoot Kubernetes applications. 

---

Agenda slide

---

**EKS Architecture**

![](images/eks-arch.jpg)

---

**How to setup a Kubernetes cluster**

{{% fragment %}} 1. Create Master Nodes (aka the Control Plane) {{% /fragment %}}

{{% fragment %}} 2. use *kubectl* to connect to the Control Plane {{% /fragment %}}

{{% fragment %}} 3. Install Worker nodes {{% /fragment %}}

{{% fragment %}} 4. Deploy apps and add-ons {{% /fragment %}}

{{% fragment %}} 5. ... profit? {{% /fragment %}}

