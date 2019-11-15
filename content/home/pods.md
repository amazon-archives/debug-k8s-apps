+++
title = "Pod is pending"
weight = 40
+++

### Pod lifecycle

You created a deployment with 8 replicas:

```
$ kubectl create -f hello-deployment.yaml 
deployment.apps/hello created
```

<div class="fragment" data-fragment-index="1" align=left>
<p>Or scaled an existing deployment to 8 replicas:</p>
</div>

<pre class="fragment bash" data-fragment-index="1" style="background-color:#f0f0f0" >
<code class="hljs shell">$ kubectl scale --replicas=8 deployment hello 
deployment.extensions/hello scaled
</code></pre>

<div class="fragment" data-fragment-index="2">
<p>Deployment shows only 4 replicas are available:</p>
</div>
<pre class="fragment" data-fragment-index="2" style="background-color:#f0f0f0" >
<code class="hljs shell">$ kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
hello   4/8     8            4           23s</code></pre>

---


### List pods

```
$ kubectl get pods

. . .

=======
NAME                     READY   STATUS    RESTARTS   AGE
hello-6d4fbd5d76-9xqxg   1/1     Running   0          5s
hello-6d4fbd5d76-brv7k   0/1     Pending   0          5s
hello-6d4fbd5d76-hbf8h   0/1     Pending   0          5s
hello-6d4fbd5d76-jdzlw   1/1     Running   0          5s
hello-6d4fbd5d76-jqsfk   0/1     Pending   0          5s
hello-6d4fbd5d76-k29gb   1/1     Running   0          5s
hello-6d4fbd5d76-vjr62   0/1     Pending   0          5s
hello-6d4fbd5d76-z69pp   1/1     Running   0          5s
```

---


### Multiple reasons for pending pod

- Not enough resources in the cluster
  - CPU, memory, port
- Node security group does not have an ingress rule with 443 port access
- Not enough IP addresses
- Ensure all nodes are healthy

---


### Describe the pod

```
$ kubectl describe pod/hello-6d4fbd5d76-brv7k
```

Shows the output:

```
. . .

Events:
  Type     Reason             Age                From                Message
  ----     ------             ----               ----                -------
  Warning  FailedScheduling   42s (x2 over 42s)  default-scheduler   0/4 nodes are available: 4 Insufficient cpu.
```

Events are only visible on pods, not on Deployments, ReplicaSet, Job, or any other resource that created pod.

---

### Get all events

```
$ kubectl get events 
LAST SEEN   TYPE      REASON             KIND   MESSAGE
2m57s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
2m57s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
2m57s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
2m57s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
```

---


### Only the warning events

```
$ kubectl get events --field-selector type=Warning
```

---

### Events only for the pod

```
$ kubectl get events --field-selector involvedObject.kind=Pod,involvedObject.name=hello-6d4fbd5d76-brv7k
LAST SEEN   TYPE      REASON             KIND   MESSAGE
4m41s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
```

---

### Sort by timestamp

```
$ kubectl get events --sort-by='.lastTimestamp'
```

---

### Memory/CPU requirements of pod

```
$ kubectl describe deployments/hello
```

Output:

```
  Containers:
   hello:
    Image:      nginx:latest
    Port:       <none>
    Host Port:  <none>
    Limits:
      cpu:     2
      memory:  2000Mi
    Requests:
      cpu:        2
      memory:     2000Mi
    Environment:  <none>
```

---

### Request and limit

Default CPU request is `200m` and none on memory. There are no limits. 

```
1000m (milicores) = 1 core = 1 CPU = 1 AWS vCPU
``` 

So, that means:

```
100m cpu = 0.1 cpu
```

In this case, CPU request and limits have been specified to `2` and memory to `2GB`. So we need 8 blocks of 2 CPU and 2 GB memory.

---

### Memory/CPU in cluster

```
$ kubectl top nodes
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get nodes.metrics.k8s.io)
```

---

### Install metrics-server


```
$ curl -OL https://github.com/kubernetes-sigs/metrics-server/archive/v0.3.6.tar.gz
$ tar xzvf v0.3.6.tar.gz
$ kubectl create -f metrics-server-0.3.6/deploy/1.8+/
```

---

### Confirm Metrics API is available

```
$ kubectl get apiservice v1beta1.metrics.k8s.io
NAME                     SERVICE                      AVAILABLE   AGE
v1beta1.metrics.k8s.io   kube-system/metrics-server   True        11d
```

---

### Memory/CPU for nodes

```
$ kubectl top nodes
NAME                                           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
ip-192-168-28-108.us-west-2.compute.internal   28m          0%     410Mi           2%        
ip-192-168-48-190.us-west-2.compute.internal   33m          0%     363Mi           2%        
ip-192-168-51-148.us-west-2.compute.internal   29m          0%     338Mi           2%        
ip-192-168-64-166.us-west-2.compute.internal   32m          0%     395Mi           2% 
```

---

### Capacity and allocatable memory for each node

_capacity_ memory:

```
$ kubectl get no -o json | jq -r '.items | sort_by(.status.capacity.memory)[]|[.metadata.name,.status.capacity.memory]| @tsv'
ip-192-168-28-108.us-west-2.compute.internal  15950552Ki
ip-192-168-48-190.us-west-2.compute.internal  15950552Ki
ip-192-168-51-148.us-west-2.compute.internal  15950552Ki
ip-192-168-64-166.us-west-2.compute.internal  15950552Ki
```

_allocatable_ memory:

```
$ kubectl get no -o json | jq -r '.items | sort_by(.status.allocatable.memory)[]|[.metadata.name,.status.allocatable.memory]| @tsv'
ip-192-168-28-108.us-west-2.compute.internal  15848152Ki
ip-192-168-48-190.us-west-2.compute.internal  15848152Ki
ip-192-168-51-148.us-west-2.compute.internal  15848152Ki
ip-192-168-64-166.us-west-2.compute.internal  15848152Ki
```

---

### How is allocatable calculated?

```
[Allocatable] = [Node Capacity] - [Kube-Reserved] - [System-Reserved] - [Hard-Eviction-Threshold]
```

{{% fragment %}}Explained at https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/node-allocatable.md. {{% /fragment %}}

---

### Capacity and allocatable CPU

_capacity_ CPU:

```
$ kubectl get no -o json | jq -r '.items | sort_by(.status.capacity.cpu)[]|[.metadata.name,.status.capacity.cpu]| @tsv'
ip-192-168-28-108.us-west-2.compute.internal  4
ip-192-168-48-190.us-west-2.compute.internal  4
ip-192-168-51-148.us-west-2.compute.internal  4
ip-192-168-64-166.us-west-2.compute.internal  4
```

_allocatable_ CPU:

```
$ kubectl get no -o json | jq -r '.items | sort_by(.status.allocatable.cpu)[]|[.metadata.name,.status.allocatable.cpu]| @tsv'
ip-192-168-28-108.us-west-2.compute.internal  4
ip-192-168-48-190.us-west-2.compute.internal  4
ip-192-168-51-148.us-west-2.compute.internal  4
ip-192-168-64-166.us-west-2.compute.internal  4
```

So, there is enough memory and CPU. Why the pods are not getting scheduled?

---

### kubeReserved and evictionHard need to be set explicitly

EKS AMI now sets a minimum `evictionHard` and `kubeReserved` values: https://github.com/awslabs/amazon-eks-ami/pull/350.

Alternatively, you can set these values using eksctl https://eksctl.io/usage/customizing-the-kubelet/.

---

### Cluster Autoscaler

Serves two purpose:

- Pods fail to run due to insufficient resources
- Recycle nodes that are underutilized for an extended period of time


Let's install it!

---

### IAM policy for Cluster Autoscaler

Create IAM policy with autoscaling permissions and attach to the worker node IAM roles.

Create IAM policy:


```
$ aws iam create-policy --policy-name AmazonEKSAutoscalingPolicy --policy-document file://../../resources/manifests/autoscaling-policy.json
{
    "Policy": {
        "PolicyName": "AmazonEKSAutoscalingPolicy",
        "PolicyId": "ANPARKOFJSCVVWD4MQEKB",
        "Arn": "arn:aws:iam::091144949931:policy/AmazonEKSAutoscalingPolicy",
        . . .
    }
}
```

Attach policy to the IAM role:

```
$ ROLE_NAME=$(aws iam list-roles \
  --query \
  'Roles[?contains(RoleName,`debug-k8s-nodegroup`)].RoleName' --output text)
$ aws iam attach-role-policy \
  --role-name $ROLE_NAME \
  --policy-arn arn:aws:iam::091144949931:policy/AmazonEKSAutoscalingPolicy
```

---

### Auto-discovery of ASG by CA

Setup auto discovery of Auto Scaling Groups by Cluster Autoscaler by attaching tags to the nodegroup:

```
$ ASG_NAME=$(aws autoscaling describe-auto-scaling-groups \
  --query \
  'AutoScalingGroups[?contains(AutoScalingGroupName,`debug-k8s-nodegroup`)].AutoScalingGroupName' --output text)
$ aws autoscaling create-or-update-tags \
  --tags \
  ResourceId=$ASG_NAME,ResourceType=auto-scaling-group,Key=k8s.io/cluster-autoscaler/enabled,Value=something,PropagateAtLaunch=true \
  ResourceId=$ASG_NAME,ResourceType=auto-scaling-group,Key=k8s.io/cluster-autoscaler/debug-k8s,Value=something,PropagateAtLaunch=true
```

---

### Create Cluster Autoscaler:


```
$ CA_FILE=cluster-autoscaler-autodiscover.yaml
$ curl -o ${CA_FILE} https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
$ sed -i -e 's/<YOUR CLUSTER NAME>/debug-k8s/' ${CA_FILE}
$ kubectl create -f ${CA_FILE}
```

---

### Cluster Autoscaler logs

```
$ kubectl logs -f deployment/cluster-autoscaler -n kube-system
```

Shows the output:

```
I1113 01:40:00.754350       1 scale_up.go:263] Pod default/hello-6d4fbd5d76-hbf8h is unschedulable
I1113 01:40:00.754358       1 scale_up.go:263] Pod default/hello-6d4fbd5d76-vjr62 is unschedulable
I1113 01:40:00.754365       1 scale_up.go:263] Pod default/hello-6d4fbd5d76-brv7k is unschedulable
I1113 01:40:00.754371       1 scale_up.go:263] Pod default/hello-6d4fbd5d76-jqsfk is unschedulable
I1113 01:40:00.754407       1 scale_up.go:300] Upcoming 0 nodes
I1113 01:40:00.754416       1 scale_up.go:335] Skipping node group eksctl-debug-k8s-nodegroup-ng-bb0efd30-NodeGroup-H77X21MZFGGH - max size reached
I1113 01:40:00.754426       1 scale_up.go:411] No expansion options
```

---

### ASG Limits

Update Autoscaling Group limits:

```
$ aws autoscaling update-auto-scaling-group --auto-scaling-group-name $ASG_NAME --max-size 8
```

Cluster Autoscaler logs are updated:

```
I1113 01:40:11.009046       1 scale_up.go:263] Pod default/hello-6d4fbd5d76-jqsfk is unschedulable
I1113 01:40:11.009052       1 scale_up.go:263] Pod default/hello-6d4fbd5d76-hbf8h is unschedulable
I1113 01:40:11.009057       1 scale_up.go:263] Pod default/hello-6d4fbd5d76-vjr62 is unschedulable
I1113 01:40:11.009062       1 scale_up.go:263] Pod default/hello-6d4fbd5d76-brv7k is unschedulable
I1113 01:40:11.009098       1 scale_up.go:300] Upcoming 0 nodes
I1113 01:40:11.009322       1 waste.go:57] Expanding Node Group eksctl-debug-k8s-nodegroup-ng-bb0efd30-NodeGroup-H77X21MZFGGH would waste 50.00% CPU, 93.58% Memory, 71.79% Blended
I1113 01:40:11.009346       1 scale_up.go:418] Best option to resize: eksctl-debug-k8s-nodegroup-ng-bb0efd30-NodeGroup-H77X21MZFGGH
I1113 01:40:11.009356       1 scale_up.go:422] Estimated 4 nodes needed in eksctl-debug-k8s-nodegroup-ng-bb0efd30-NodeGroup-H77X21MZFGGH
I1113 01:40:11.009374       1 scale_up.go:501] Final scale-up plan: [{eksctl-debug-k8s-nodegroup-ng-bb0efd30-NodeGroup-H77X21MZFGGH 4->8 (max: 8)}]
```

---

### Check pods

```
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
hello-6d4fbd5d76-9xqxg   1/1     Running   0          6m30s
hello-6d4fbd5d76-brv7k   1/1     Running   0          6m30s
hello-6d4fbd5d76-hbf8h   1/1     Running   0          6m30s
hello-6d4fbd5d76-jdzlw   1/1     Running   0          6m30s
hello-6d4fbd5d76-jqsfk   1/1     Running   0          6m30s
hello-6d4fbd5d76-k29gb   1/1     Running   0          6m30s
hello-6d4fbd5d76-vjr62   1/1     Running   0          6m30s
hello-6d4fbd5d76-z69pp   1/1     Running   0          6m30s
```

---

### Similar usecase

```
$ kubectl get pods -l app=mnist,type=inference
NAME                             READY   STATUS    RESTARTS   AGE
mnist-inference-cd78cfd5-hcvfd   0/1     Pending   0          3m48s
```

---

### Similar analysis

Get details about the pod:

```
$ kubectl describe pod mnist-inference-cd78cfd5-hcvfd

. . .

Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  3s (x8 over 5m32s)  default-scheduler  0/2 nodes are available: 2 Insufficient nvidia.com/gpu.
```

---

Need to create a cluster with more GPUs.

