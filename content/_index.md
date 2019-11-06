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

<style type="text/css">
  .reveal p {
    text-align: left;
  }
  .reveal ul {
    display: block;
  }
  .reveal ol {
    display: block;
  }
</style>

# Tutorial: Debug Your Kubernetes Apps
<section>
  Arun Gupta & Re Alvarez Parmar, Amazon
</section>

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

