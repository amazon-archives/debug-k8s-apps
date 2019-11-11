+++
weight=8
+++

# How to find problems?

---

### Monitoring tools
- [Prometheus](https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html) + [Grafana](https://eksworkshop.com/monitoring/)
- [CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights-EKS.html)
- Kubernetes [Dashboard](https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html)
- Commercial solutions (e.g., Datadog, Dynatrace)

---

### What to monitor?

<section>
  <ul class="fragment fade-in-then-semi-out" data-fragment-index="1">
    <li>Nodes</li>
    <ul><li>CPU/Memory/Disk/Network</li></ul>
  </ul>

  <ul class="fragment fade-in-then-semi-out" data-fragment-index="2">
    <li>Pods/Deployments</li>
    <ul>
      <li>Pod count</li>
      <li>Container health checks</li>
    </ul>
  </ul>

  <ul class="fragment fade-in-then-semi-out" data-fragment-index="3">
    <li>coreDNS</li>
    <ul>
      <li>Latency</li>
  </ul>


</section> 

---

<section data-markdown>
### Key things to monitor
  <script> 
  ### Key things to monitor
- Nodes <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="1" -->
   - CPU/Memory/Disk <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="1" -->
- Pods and Deployments <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="2" -->
   - Pod count <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="2" -->
   - Container health checks <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="2" -->
- coreDNS <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="3" -->
   - DNS latency <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="3" -->
  </script>
</section>     
