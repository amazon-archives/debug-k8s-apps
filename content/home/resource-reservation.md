+++
weight = 70
+++

{{% section %}}

## Section 7:
## Kubelet Resource Reservation

---

### Kubelet resource reservation

- Monitor kubelet on the worker node (systemd-based systems)

```
$ journalctl -u kubelet
```

- Use **kube-reserved** to reserve resources for kubelet, container runtime & node problem detector

```
--kube-reserved=[cpu-100m][,][memory=100Mi][,]
[ephemeral-storage=1Gi][,][pid=1000]
```

- Use **system-reserved** to reserve resources for system daemons liks sshd, udev, kernel

```
--system-reserved=[cpu-100m][,][memory=100Mi][,]
[ephemeral-storage=1Gi][,][pid=1000]
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

- Use resource quotas on namespaces:

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

{{% note %}}
A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per namespace. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that project.
{{% /note %}}

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
By default, containers run with unbounded compute resources on a Kubernetes cluster, or in a namespace's resource quota. Limit Range is a policy to constrain resource by Pod or Container in a namespace.
{{% /note %}}

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

{{% /section %}}
