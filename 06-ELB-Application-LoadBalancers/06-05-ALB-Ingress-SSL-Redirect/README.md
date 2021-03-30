# AWS ALB Load Balancer Controller - Implement HTTP to HTTPS Redirect

## Step-01: Add annotations related to SSL Redirect

- Redirect from HTTP to HTTPS
- alb.ingress.kubernetes.io/actions.${action-name} - Provides a method for configuring custom actions on a listener, such as for Redirect Actions.
- The `action-name` in the annotation must match the `serviceName` in the ingress rules, and `servicePort` must be `use-annotation`.
- **Reference for Custom Actions:** https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/main/docs/guide/ingress/annotations.md#actions
- The Actions(Listeners->Rules) supported on AWS ALB through console are the ones which will be configured as annotatiions in Ingress manifest file.

### Change-1: Add the Custom Action Annotation

```yml
# SSL Redirect Setting
alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
```

- **SSL Redirect JSON**

```json
{
   "Type":"redirect",
   "RedirectConfig":{
      "Protocol":"HTTPS",
      "Port":"443",
      "StatusCode":"HTTP_301"
   }
```

### Change-2: Add the following Path as first ingress rule in the Rules section

```yml
- path: /* # SSL Redirect Setting
  backend:
    serviceName: ssl-redirect
    servicePort: use-annotation
```

## Step-02: Deploy all manifests and test

- **Deploy**

```
# Deploy
kubectl apply -f kube-manifests/
```

- **Verify**
  - Load Balancer - Listeneres (Verify both 80 & 443)
  - Load Balancer - Rules (Verify both 80 & 443 listeners)
  - Target Groups - Group Details (Verify Health check path)
  - Target Groups - Targets (Verify all 3 targets are healthy)
  - Verify ingress controller from kubectl

```
kubectl get ingress
```

## Step-04: Access Application using newly registered DNS Name

- **Access Application**

```
# HTTP URLs (Should Redirect to HTTPS)
http://ssldemo.awseksstudy.com/app1/index.html
http://ssldemo.awseksstudy.com/app2/index.html
http://ssldemo.awseksstudy.com/usermgmt/health-status

# HTTPS URLs
https://ssldemo.awseksstudy.com/app1/index.html
https://ssldemo.awseksstudy.com/app2/index.html
https://ssldemo.awseksstudy.com/usermgmt/health-status
```

## Step-05: Clean Up

### Delete Manifests

```
kubectl delete -f kube-manifests/
```

### Delete Route53 Record Set

- Delete Route53 Record we created (ssldemo.kubeoncloud.com)

## Step-06: Annotation Reference

- Discuss about location where that Annotation can be placed (Ingress or Service)
- https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/main/docs/guide/ingress/annotations.md
