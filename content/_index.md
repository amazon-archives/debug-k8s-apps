+++
title = "Tutorial: Debug Your Kubernetes Apps - Arun Gupta & Re Alvarez Parmar, Amazon Web Services"
outputs = ["Reveal"]

[favicon]
src = "images/favicon.ico"

[logo]
src="images/kubecon-slide-header.png"

[reveal_hugo]
theme = "white"
margin = 0.1
width = "100%"
height = "89%"
slide_number = true
controls = true
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
    max-height: 65%;
    max-width: auto;
    margin-left: auto;
    margin-right: auto;
    tex-align: center;
    display: block;
  }
</style>

<section data-background-image="images/kubecon-slide-theme.png"
data-background-size=cover data-background-color="#000000">
</section>

---
<section data-background-image="images/kubecon-slide-title.png"
data-background-size=contain data-background-color="#000000" >
</section>

---

<center>
<img src="images/dilbert-k8s.jpeg"/>
</center>

---

<center>
<img src="images/k8s-heisenberg.png"/>
</center>

{{% note %}}
In this presentation we will learn how to troubleshoot Kubernetes applications. 
{{% /note %}}

---

### Topics

1. Cluster Design

2. Networking

3. Kubectl

4. Pods

5. Load Balancing & Ingress

6. Monitoring


7. Resource Reservation

8. StatefulSets



