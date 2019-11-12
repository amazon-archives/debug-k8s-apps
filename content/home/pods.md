+++
weight = 3
+++

# Pod is pending

---

You created a deployment:

```
kubectl create -f hello-deployment.yaml 
deployment.apps/hello created
```

and scaled to 8 replicas:

```
$ kubectl scale --replicas=8 deployment hello 
deployment.extensions/hello scaled
```

Deployment shows only 4 replicas are available:

```
$ kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
hello   4/8     8            4           23s
```

---

This is matched by the output of `get pods`:

```
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
hello-67c5c968fd-2s4n7   1/1     Running   0          64m
hello-67c5c968fd-5lzxw   0/1     Pending   0          64m
hello-67c5c968fd-894z6   1/1     Running   0          64m
hello-67c5c968fd-gv9cw   0/1     Pending   0          64m
hello-67c5c968fd-lwww8   0/1     Pending   0          64m
hello-67c5c968fd-p8mxd   1/1     Running   0          64m
hello-67c5c968fd-vlnsd   1/1     Running   0          64m
hello-67c5c968fd-wj6j8   0/1     Pending   0          64m
```

---

Multiple reasons:

- Not enough resources in the cluster
  - CPU, memory, port
- Security group does not have an ingress rule with 443 port access
- Ensure all nodes are healthy

---

Describe the pod:

```
kubectl describe pod/hello-67c5c968fd-5lzxw
```

Shows the output:

```
. . .

Events:
  Type     Reason            Age                    From               Message
  ----     ------            ----                   ----               -------
  Warning  FailedScheduling  107s (x172 over 147m)  default-scheduler  0/4 nodes are available: 4 Insufficient cpu.
```

Events are only visible on pods, not on Deployments, ReplicaSet, Job, or any other resource that created pod.

---

Alternatively, get all events:

```
$ kubectl get events 
LAST SEEN   TYPE      REASON             KIND   MESSAGE
2m57s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
2m57s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
2m57s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
2m57s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
```

---

Or only the warning events:

```
kubectl get events --field-selector type=Warning
```

---

Let's get events only for the pod:

```
$ kubectl get events --field-selector involvedObject.kind=Pod,involvedObject.name=hello-67c5c968fd-5lzxw
LAST SEEN   TYPE      REASON             KIND   MESSAGE
4m41s       Warning   FailedScheduling   Pod    0/4 nodes are available: 4 Insufficient cpu.
```

---

Sort by timestamp:

```
$ kubectl get events --sort-by='.lastTimestamp'
```

---

Check memory/CPU requirements of pod:

```
kubectl describe deployments/hello
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

Default CPU request is `200m` and none on memory. There are no limits. 

1000m (milicores) = 1 core = 1 CPU = 1 AWS vCPU.

100m cpu = 0.1 cpu

In this case, CPU request and limits have been specified to `2` and memory to `2GB`. So we need 8 blocks of 2 CPU and 2 GB memory.

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

Get available memory on each node:

```
kubectl get no -o json | jq -r '.items | sort_by(.status.capacity.memory)[]|[.metadata.name,.status.capacity.memory]| @tsv'
ip-192-168-28-108.us-west-2.compute.internal	15950552Ki
ip-192-168-48-190.us-west-2.compute.internal	15950552Ki
ip-192-168-51-148.us-west-2.compute.internal	15950552Ki
ip-192-168-64-166.us-west-2.compute.internal	15950552Ki
```

And allocatable memory:

```
kubectl get no -o json | jq -r '.items | sort_by(.status.allocatable.memory)[]|[.metadata.name,.status.allocatable.memory]| @tsv'
ip-192-168-28-108.us-west-2.compute.internal	15950552Ki
ip-192-168-48-190.us-west-2.compute.internal	15950552Ki
ip-192-168-51-148.us-west-2.compute.internal	15950552Ki
ip-192-168-64-166.us-west-2.compute.internal	15950552Ki
```

---

How is **allocatable** calculated?

```
[Allocatable] = [Node Capacity] - [Kube-Reserved] - [System-Reserved] - [Hard-Eviction-Threshold]
```

Explained at https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/node-allocatable.md.

---


And do the same for capacity CPU:

```
kubectl get no -o json | jq -r '.items | sort_by(.status.capacity.cpu)[]|[.metadata.name,.status.capacity.cpu]| @tsv'
ip-192-168-28-108.us-west-2.compute.internal	4
ip-192-168-48-190.us-west-2.compute.internal	4
ip-192-168-51-148.us-west-2.compute.internal	4
ip-192-168-64-166.us-west-2.compute.internal	4
```

And allocatable CPU:

```
kubectl get no -o json | jq -r '.items | sort_by(.status.allocatable.cpu)[]|[.metadata.name,.status.allocatable.cpu]| @tsv'
ip-192-168-28-108.us-west-2.compute.internal	4
ip-192-168-48-190.us-west-2.compute.internal	4
ip-192-168-51-148.us-west-2.compute.internal	4
ip-192-168-64-166.us-west-2.compute.internal	4
```

So, there is enough memory but not CPU.

EKS AMI now sets a minimum `evictionHard` and `kubeReserved` values: https://github.com/awslabs/amazon-eks-ami/pull/350.

Alternatively, you can set these values using eksctl https://eksctl.io/usage/customizing-the-kubelet/.

---

Cluster autoscaler serves two purpose:

- Pods fail to run due to insufficient resources
- Recycle nodes that are underutilized for an extended period of time


Let's install it!

---

Create IAM policy with autoscaling permissions and attach to the worker node IAM roles:

```
aws iam create-policy --policy-name AmazonEKSAutoscalingPolicy --policy-document file://../../resources/manifests/autoscaling-policy.json
{
    "Policy": {
        "PolicyName": "AmazonEKSAutoscalingPolicy",
        "PolicyId": "ANPARKOFJSCVVWD4MQEKB",
        "Arn": "arn:aws:iam::091144949931:policy/AmazonEKSAutoscalingPolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2019-11-12T22:23:14Z",
        "UpdateDate": "2019-11-12T22:23:14Z"
    }
}
```

Find IAM role:

```
ROLE_NAME=$(aws iam list-roles \
  --query \
  'Roles[?contains(RoleName,`debug-k8s-nodegroup`)].RoleName' --output text)
```

Attach policy to the IAM role:

```
aws iam attach-role-policy \
  --role-name $ROLE_NAME \
  --policy-arn arn:aws:iam::091144949931:policy/AmazonEKSAutoscalingPolicy
```

---

Find the Auto Scaling Group:

```
ASG_NAME=$(aws autoscaling describe-auto-scaling-groups \
  --query \
  'AutoScalingGroups[?contains(AutoScalingGroupName,`debug-k8s-nodegroup`)].AutoScalingGroupName' --output text)
```

---

Attach tags to the nodegroup:

```
aws autoscaling create-or-update-tags \
  --tags \
  ResourceId=$ASG_NAME,ResourceType=auto-scaling-group,Key=k8s.io/cluster-autoscaler/enabled,Value=something,PropagateAtLaunch=true \
  ResourceId=$ASG_NAME,ResourceType=auto-scaling-group,Key=k8s.io/cluster-autoscaler/debug-k8s,Value=something,PropagateAtLaunch=true
```

---

Now, create Cluster Autoscaler:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

---

Create horizontal pod autoscaler

```
```

---



---

```
$ kubectl get pods -l app=mnist,type=inference
NAME                             READY   STATUS    RESTARTS   AGE
mnist-inference-cd78cfd5-hcvfd   0/1     Pending   0          3m48s
```


```
kubectl describe pod mnist-inference-cd78cfd5-hcvfd

. . .

Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  3s (x8 over 5m32s)  default-scheduler  0/2 nodes are available: 2 Insufficient nvidia.com/gpu.
```

