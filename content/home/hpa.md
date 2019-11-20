+++
title = "Horizontal Pod Autoscaling"
weight = 90
+++

{{% section %}}

## Section 9:
## Pod Scaling

---

### Connection refused to pod

Let's look at a different part of the problem now.

Pods are all scheduled but not able to meet throughput and/or latency needs.

---

Create horizontal pod autoscaler

[Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) (HPA) scales the pods in a deployment or replica set. It is implemented as a K8s API resource and a controller. The controller manager queries the resource utilization against the metrics specified in each HorizontalPodAutoscaler definition. It obtains the metrics from either the resource metrics API (for per-pod resource metrics), or the custom metrics API (for all other metrics).

Default HPA loop is 15 seconds (controlled by `--horizontal-pod-autoscaler-sync-period`)

---

Creata a Deployment and Service:


```
$ kubectl create -f resources/manifests/hpa-example.yaml
```

The index.php page performs calculations to generate CPU load. More information can be found [here](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#run-expose-php-apache-server).

---

Create a shell using a new container:

```
$ kubectl run -i --tty load-generator --image=busybox /bin/sh
```

---

Execute a loop to invoke the service:

```
while true; do wget -q -O - http://php-apache; done
```

Shows the output as:

```
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!wget: error getting response: No such file or directory
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
wget: can't connect to remote host (10.100.59.78): Connection refused
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!
```

---

Create an autoscale deployment:

```
$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

---

Get details about HPA:

```
$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          20s
```

--- 

Generate load to trigger scaling:

```
$ kubectl run -i --tty load-generator --image=busybox /bin/sh
```

Execute a while loop to generate load:

```
# while true; do wget -q -O - http://php-apache; done
```

---

Get the current status of HPA:

```
$ kubectl get hpa -w
NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%     1         10        1          20m
php-apache   Deployment/php-apache   167%/50%   1         10        4          21m
php-apache   Deployment/php-apache   128%/50%   1         10        4          22m
php-apache   Deployment/php-apache   128%/50%   1         10        8          22m
php-apache   Deployment/php-apache   128%/50%   1         10        10         22m
php-apache   Deployment/php-apache   46%/50%    1         10        10         23m
php-apache   Deployment/php-apache   0%/50%     1         10        10         25m
php-apache   Deployment/php-apache   0%/50%     1         10        1          29m
```


---

```
/ # while true; do wget -q -O - http://php-apache; done
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK! . . .
```

---

{{% /section %}}
