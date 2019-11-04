+++
weight = 20
+++

# Let's talk about HPA

---

[Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) (HPA) scales the pods in a deployment or replica set. It is implemented as a K8s API resource and a controller. The controller manager queries the resource utilization against the metrics specified in each HorizontalPodAutoscaler definition. It obtains the metrics from either the resource metrics API (for per-pod resource metrics), or the custom metrics API (for all other metrics).

Default HPA loop is 15 seconds (controlled by `--horizontal-pod-autoscaler-sync-period`)

Install metrics server:

```
helm install stable/metrics-server \
    --name metrics-server \
    --version 2.0.4 \
    --namespace metrics
```

Deploy a sample app:

```
kubectl run php-apache --generator=run-pod/v1 --image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80
```

Get all the objects:

```
kubectl get all
NAME             READY   STATUS    RESTARTS   AGE
pod/php-apache   1/1     Running   0          31s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.100.0.1      <none>        443/TCP   23h
service/php-apache   ClusterIP   10.100.150.64   <none>        80/TCP    31s
```

Watch pod stats:

```
watch kubectl top pod 
```





HPA fetches metrics from `metrics.k8s.io` that is provided by metrics server. It needs to be installed separately.

