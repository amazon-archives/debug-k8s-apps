+++
weight=5
+++

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
