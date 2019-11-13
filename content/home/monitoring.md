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
Install Prometheus

```
$ kubectl create namespace prometheus
$ helm install stable/prometheus \
    --name prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

verify Prometheus

```
$ kubectl -n prometheus get pods
prometheus-alertmanager-58c6876d4c-6p5qs              2/2     Running   0          20s
prometheus-kube-state-metrics-74b57df8bd-jksg8        1/1     Running   0          20s
prometheus-node-exporter-2nvkk                        1/1     Running   0          20s
prometheus-node-exporter-fzlts                        1/1     Running   0          20s
prometheus-node-exporter-s4nsc                        1/1     Running   0          20s
prometheus-pushgateway-c497ff84d-bxm5b                1/1     Running   0          20s
prometheus-server-796c867465-q775s                    2/2     Running   0          20s
```

---

access Prometheus

```
$ kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090
```

<center>
<img src="https://eksworkshop.com/images/prometheus-targets.png" height="300" >
</center>

---

Install Grafana

```
$ kubectl create namespace grafana
$ helm install stable/grafana \
    --name grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set adminPassword='EKS!sAWSome' \
    --set datasources."datasources\.yaml".apiVersion=1 \
    --set datasources."datasources\.yaml".datasources[0].name=Prometheus \
    --set datasources."datasources\.yaml".datasources[0].type=prometheus \
    --set datasources."datasources\.yaml".datasources[0].url=http://prometheus-server.prometheus.svc.cluster.local \
    --set datasources."datasources\.yaml".datasources[0].access=proxy \
    --set datasources."datasources\.yaml".datasources[0].isDefault=true \
    --set service.type=LoadBalancer
```

verify Grafana pods

```
$ kubectl -n grafana get pods
NAME                          READY     STATUS    RESTARTS   AGE
pod/grafana-b9697f8b5-t9w4j   1/1       Running   0          2m
```

---

Obtain Grafana ELB URL

```
$ export ELB=$(kubectl get svc \
    -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
$ echo "http://$ELB"
```

Get password for admin user

```
$ kubectl get secret --namespace grafana grafana \
      -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

---

## Community created Dashboard #3131
<center>
  <img src="https://eksworkshop.com/images/grafana-all-nodes.png" height="75%" width="75%">
</center>

---

## Community created Dashboard #3146
<center>
  <img src="https://eksworkshop.com/images/grafana-all-pods.png" height="75%" width="75%">
</center>

---

### CloudWatch Container Inights
- Collect, aggregate, and summarize metrics
- Capture logs (uses FluentD)
- Get diagnostic information, such as container restart failures

---
### Cloudwatch Container Insights components
1. CloudWatch agent daemonset
2. FluentD daemonset

```
$ kubectl get ds -n amazon-cloudwatch
NAME                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
cloudwatch-agent     3         3         3       3            3           <none>          21h
fluentd-cloudwatch   3         3         3       3            3           <none>          21h
```
---

#### CloudWatch Container Insights Dashboard

<center><img src=images/Container-insights.png height="75%" width="75%"></center>


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



