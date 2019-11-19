+++
weight = 70
+++

{{% section %}}

## Section 7:
## Going public, Load balancers and Ingress

---

### Options for exposing Kubernetes apps
- ServiceType=LoadBalancer. Uses Classic load balancer.
    - Use Network load balancer, annotate your service:
```
service.beta.kubernetes.io/aws-load-balancer-type: nlb
```
- Ingress using [alb-ingresss-controller](https://github.com/kubernetes-sigs/aws-alb-ingress-controller)
    - For internal only load balancer, annotate your service:
```
service.beta.kubernetes.io/aws-load-balancer-internal: 0.0.0.0/0

```
- [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx)

---

### If load balancer fails to deploy
- Check [tags](https://docs.aws.amazon.com/en_pv/eks/latest/userguide/network_reqs.html)
- Check service

```
kubectl describe service
```
- Check [EKS IAM Service Role](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html)

---
### Troubleshoot latency in app
- Check [CloudWatch Metrics for load balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-monitoring.html)
- Check latency within cluster
```
kubectl apply -f https://k8s.io/examples/application/shell-demo.yaml

kubectl exec -it shell-demo -- /bin/bash

root@shell-demo:/# apt-get update 
root@shell-demo:/# apt-get install curl
root@shell-demo:/# curl http://10.100.2.39 <-- IP of the service
```
- Implement [tracing](https://docs.aws.amazon.com/whitepapers/latest/microservices-on-aws/distributed-monitoring.html)
preferably with a [Service Mesh](https://aws.amazon.com/app-mesh/)


{{% /section %}}
