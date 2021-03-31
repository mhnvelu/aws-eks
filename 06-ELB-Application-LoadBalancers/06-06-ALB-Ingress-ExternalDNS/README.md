# External DNS

- External DNS is created using k8s manifests.
- **Using this External DNS k8s manifests and adding annotations in Ingress manifests related to External DNS, will allow us to add Record Set or DNS entries in Route53.**

1. Deploy External DNS
2. Use External DNS for our Application

## External DNS References

- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/alb-ingress.md
- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md
