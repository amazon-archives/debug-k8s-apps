+++
weight = 30
+++

### Kubelet resource reservation

- Monitor kubelet on the worker node

```
journalctl -u kubelet
```

- Use kube-reserved to reserve resources for kubelet, container runtime & node problem detector

```
--kube-reserved=[cpu-100m][,][memory=100Mi][,]
[ephemeral-storage=1Gi][,][pid=1000]
```

- Use system-reserved to reserve resources for system daemons liks sshd, udev, kernel
```
--system-reserved=[cpu-100m][,][memory=100Mi][,]
[ephemeral-storage=1Gi][,][pid=1000]
```

---

### Kubelet resource reservation for Amazon EKS using User Data

```
ADD EXAMPLE
```

---

### Kubelet resource reservation for Amazon EKS using eksctl

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: myeks
  region: us-east-1

nodeGroups:
  - name: myng
    instanceType: m5.xlarge
    desiredCapacity: 4
    kubeletExtraConfig:
        kubeReserved:
            cpu: "300m"
            memory: "300Mi"
            ephemeral-storage: "1Gi"
        kubeReservedCgroup: "/kube-reserved"
        systemReserved:
            cpu: "300m"
            memory: "300Mi"
            ephemeral-storage: "1Gi"
        evictionHard:
            memory.available:  "200Mi"
            nodefs.available: "10%"
        featureGates:
            DynamicKubeletConfig: true
            RotateKubeletServerCertificate: true # has to be enabled, otherwise it will be disabled
```

---

### Avoid oversubscription

- Use resource limits and requests

    ```
    Pod.sepc.containers.resources: 
                          limits: 
                            memory: 128Mi 
                            cpu: 500m 
                          requests:
                            memory: 64Mi 
                            cpu: 250m
    ```
- Use resource quotas on namespaces
    ```
    apiVersion: v1 
    kind: ResourceQuota 
    metadata: 
      name: resource-quota 
        namespace: quota-ns
    spec: 
      hard: 
          limits.cpu: 2
    ```

---

Once the quota on the namespace is full, newer pods will fail

```
$ kubectl  get events
...
Error creating: pods "nginx-fb556779d-xmsgv" is forbidden: 
exceeded quota: resource-quota, requested: limits.cpu=500m, used: limits.cpu=2, limited: limits.cpu=2
```

---

### Use **LimitRange** to set default limit

```
apiVersion: v1 
kind: LimitRange 
metadata: 
  name: limit-range-example 
    namespace: limit-range-example 
    spec: 
      limits: 
        - default: 
            memory: 512Mi 
            cpu: 1 
          defaultRequest: 
            memory: 256Mi 
            cpu: 500m 
          type: Container
```

{{% note %}}
“When you are using quotas on a namespace, one requirement is that every container in the namespace must have resource limits and requests defined. Sometimes this requirement can cause complexity and make it more difficult to work quickly with Kubernetes. Specifying resource limits correctly, while an essential part of preparing an application for production, does add additional overhead when, for example, using Kubernetes as a platform for development or testing workloads.”
{{% note %}}

---

```
$ kubectl describe limitranges 
Name:       limit-mem-cpu-per-container
Namespace:  limitrange
Type        Resource  Min   Max   Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---   ---------------  -------------  -----------------------
Container   cpu       100m  800m  110m             700m           -
Container   memory    99Mi  1Gi   111Mi            900Mi          -
```

---

### Limits scenarios 

(container limits and requests override Limitranges)

1. If container defines both *limits* and *requests*, *LimitRanges* will have no effect
2. If container has no *limits* or *requests*, *LimitRanges* will be inherited
3. If container has *requests* but not *limits*, *limits* will be inherited from *LimitRanges*
4. If container has *limits* defined, then *requests* = *limits*, *LimitRanges* will have no effect

