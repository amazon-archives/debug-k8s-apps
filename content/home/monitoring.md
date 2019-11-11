+++
weight=8
+++

# How to find problems?

---

### 🎛 Monitoring tools
- [Prometheus](https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html) + [Grafana](https://eksworkshop.com/monitoring/)
- [CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights-EKS.html)
- Kubernetes [Dashboard](https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html)
- Commercial solutions (e.g., Datadog, Dynatrace, Sysdig, etc.)

{{% note %}}
Prometheus provides metrics storage. Grafana provides dashboards. 
{{% /note %}}

---

<section data-markdown>
### Key things to monitor
  <script> 
  ### Key things to monitor
- Nodes <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="1" -->
   - CPU/memory usage, disk pressure & network <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="1" -->
- Pods, deployments and services <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="2" -->
   - Pod count <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="2" -->
   - Container health checks <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="2" -->
- coreDNS <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="3" -->
   - DNS latency <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="3" -->
  </script>
</section>

---

### Container monitoring options
- Readiness probe
- Liveness probe 
- Tracing (OpenTracing, AWS X-ray)
- Process health 
- Process metrics
- Process logs

{{% note %}}
A good cloud native container must provide APIs for runtime to monitor the health, if checks fail, an action should be triggered.
{{% /note %}}

---
