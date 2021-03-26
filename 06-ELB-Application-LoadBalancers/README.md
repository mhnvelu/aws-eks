# Load Balancing workloads on EKS using AWS Application Load Balancer

## ALB

- ALB is a advanced, next-gen Load Balancer in AWS.
- Supports Path based routing.
- Supports Host based routing.
- Support for routing based on fields in the request (HTTP Headers, HTTP Methods, Query Parameters, Source IP Address).
- Support for redirecting requests from one URL to another.
- Support for returning custom HTTP response.
- Support for registering Lambda functions as targets.
- Support for the LB to authenticate users of your applications through their corporate or social identities before routing the requests.
- Support for containerized applications (AWS ECS).
- Support for monitoring the health of each service independently, as health checks are defined at the target group level.
- Support for registering targets by IP address, including targets outside the VPC for the LB. This is leveraged in EKS Fargate.

## AWS Load Balancer Controller (AWS LBC)

- AWS LBC triggers the creation of ALB and necessary supporting resources whenever an Ingress resource is created on the cluster with `kubernetes.io/ingress.class:alb` annotation.
- AWS LBC supports 2 traffic modes.
  - Instance Mode
    - Registers managed worker nodes within the EKS cluster as targets for the ALB.
    - Traffic reaching the ALB is `routed to NodePort of your service` and then proxied to the pods.
    - This is the `default traffic mode`. It can be used for both EC2 managed node groups and Fargate.
    - We can explicitly specify it with the `alb.ingress.kubernets.io/target-type:instance` annotation.
  - IP Mode
    - Registers Pods as target for the ALB.
    - Traffic reaching the `ALB is directly routed to Pods` for your service.
    - We must specify the `alb.ingress.kubernets.io/target-type:ip` annotation to use this traffic mode.

### How AWS Load Balancer Controller (AWS LBC) works?

https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/main/docs/how-it-works.md

## Topics

- We will be looking in to this topic very extensively in a step by step and module by module model.
- The below will be the list of topics covered as part of AWS Load Balancer Controller

| S.No | Topic Name                                                |
| ---- | --------------------------------------------------------- |
| 1.   | AWS Load Balancer Controller Installation                 |
| 2.   | AWS Load Balancer Controller Basics                       |
| 3.   | AWS Load Balancer Controller Context Path based Routing   |
| 4.   | AWS Load Balancer Controller SSL                          |
| 5.   | AWS Load Balancer Controller SSL Redirect (HTTP to HTTPS) |
| 6.   | AWS Load Balancer Controller External DNS                 |

## References:

- Good to refer all the below for additional understanding.

### AWS Load Balancer Controller Pre-requisite Setup - References:

- https://github.com/kubernetes-sigs/aws-load-balancer-controller#readme

- How it works?
  - https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/main/docs/how-it-works.md
- Examples:
  - https://github.com/kubernetes-sigs/aws-load-balancer-controller/tree/master/docs/examples/2048

### AWS Load Balancer Controller Annotations Reference

- https://github.com/kubernetes-sigs/aws-load-balancer-controller/tree/master/docs/guide/ingress

### eksctl getting started

- https://eksctl.io/introduction/#getting-started

### External DNS

- https://github.com/kubernetes-sigs/external-dns
- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/alb-ingress.md
- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md
