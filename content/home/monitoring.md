+++
weight = 80
+++

{{% section %}}

## Section 8.
## Finding Problems

---

### What metrics to monitor
- Nodes
- Cluster components
- Add-ons
- Apps

---
### Key metrics 
- Nodes
  - CPU, memory, network, disk
- Cluster Components
  - etcd
  - coredns latency 
- Add-ons
  - Cluster auto scaler
  - Ingress controller
- Apps
  - Container CPU/memory/network utilization
  - Error rates 
  - App specific metrics

---
###   Monitoring tools
- [Prometheus](https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html) + [Grafana](https://eksworkshop.com/monitoring/)
- [CloudWatch Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights-EKS.html)
- Kubernetes [Dashboard](https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html)
-    - Commercial solutions (e.g., Datadog, Dynatrace, Sysdig, etc.)

{{% note %}}
Prometheus provides metrics storage. Grafana provides dashboards.
{{% /note %}}

---

### Install Prometheus

```
$ kubectl create namespace prometheus
$ helm install stable/prometheus \
    --name prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

--- 

### Verify Prometheus

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

### Access Prometheus

```
$ kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090
```

<center>
<img src="https://eksworkshop.com/images/prometheus-targets.png" />
</center>

---

### Install Grafana

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

---

### Verify Grafana pods

```
$ kubectl -n grafana get pods
NAME                          READY     STATUS    RESTARTS   AGE
pod/grafana-b9697f8b5-t9w4j   1/1       Running   0          2m
```

---

### Get Grafana ELB URL

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

<img src="https://eksworkshop.com/images/grafana-all-nodes.png" />

---

## Community created Dashboard #3146

<img src="https://eksworkshop.com/images/grafana-all-pods.png" />

---

### Important log files

- Node logs
- Kubernetes control plane logs
  - API Server
  - Controller Manager
  - Scheduler
- Kubernetes audit logs
- App container logs

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

![](images/Container-insights-pod.png)

---
### Using CloudWatch Logs Insights
- Container Insights collects metrics by using performance log events
- Durable, logs stored in CloudWatch Logs
- CloudWatch Logs Insights queries for additional views of your container data

---
#### Query container logs and Kubernetes metrics
<img src=images/Container-insights-logs.jpg />

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

{{% /section %}}
