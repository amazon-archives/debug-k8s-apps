+++
weight = 3
+++

# Pod is pending

---

$ kubectl get pods -l app=mnist,type=inference
NAME                             READY   STATUS    RESTARTS   AGE
mnist-inference-cd78cfd5-hcvfd   0/1     Pending   0          3m48s

---

Multiple reasons:

- Not enough resources in the cluster
  - CPU, memory, port
- Security group does not have an ingress rule with 443 port access
- Ensure all nodes are healthy

---

Check the memory/CPU requirements of pod

```
kubectl describe pod/hello
```

---

Check memory/CPU available in cluster:

```
$ kubectl top nodes
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get nodes.metrics.k8s.io)
```

---

Install metrics-server:


```
curl -OL https://github.com/kubernetes-sigs/metrics-server/archive/v0.3.6.tar.gz
tar xzvf v0.3.6.tar.gz
kubectl create -f metrics-server-0.3.6/deploy/1.8+/
```

---

Get memory/CPU for nodes:

```
$ kubectl top nodes
NAME                                           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
ip-192-168-28-108.us-west-2.compute.internal   28m          0%     410Mi           2%        
ip-192-168-48-190.us-west-2.compute.internal   33m          0%     363Mi           2%        
ip-192-168-51-148.us-west-2.compute.internal   29m          0%     338Mi           2%        
ip-192-168-64-166.us-west-2.compute.internal   32m          0%     395Mi           2% 
```

---

Create cluster autoscaler


---



---



---



---


```
kubectl describe pod mnist-inference-cd78cfd5-hcvfd

. . .

Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  3s (x8 over 5m32s)  default-scheduler  0/2 nodes are available: 2 Insufficient nvidia.com/gpu.
```

