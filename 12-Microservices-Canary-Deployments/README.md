# Microservices Canary Deployments

## Step-01: Introduction

- Incremental rollouts
- A newer version of the application is slowly deployed to the K8s cluster while getting a very small amount of live traffic.
- A subset of live users are connecting to a new version while the rest are still using the previous version.
- Using canaries, we can deduct deployment issues very early while they affect only small subset of users.
- If we encounter any issues with a canary, the production version is still present, and all traffic can simply be reverted to it.

### Usecase Description

- User Management **getNotificationAppInfo** will call Notification service V1 and V2 versions.
- We will distribute traffic between V1 and V2 versions of Notification service as per our choice based on Replicas

| NS V1 Replicas | NS V2 Replicas | Traffic Distribution          |
| -------------- | -------------- | ----------------------------- |
| 4              | 0              | 100% traffic to NS V1 Version |
| 3              | 1              | 25% traffic to NS V2 Version  |
| 2              | 2              | 50% traffic to NS V2 Version  |
| 1              | 3              | 75% traffic to NS V2 Version  |
| 0              | 4              | 100% traffic to NS V2 Version |

- In our demo, we are going to distribute 50% traffic to each version (V1 and V2).
- NS V1 - 2 replicas and NS V2 - 2 replicas
- We are going to depict one Microservice calling other Microservices with different versions in AWS X-Ray

### List of Docker Images used in this section

| Application Name              | Docker Image Name                                                     |
| ----------------------------- | --------------------------------------------------------------------- |
| User Management Microservice  | stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay          |
| Notifications Microservice V2 | stacksimplify/kube-notifications-microservice:4.0.0-AWS-XRay          |

## Step-02: Pre-requisite: AWS RDS Database, ALB Ingress Controller, External DNS & X-Ray Daemon

### AWS RDS Database

- Created AWS RDS Database as part of section [04-EKS-Storage-with-RDS-Database](/04-EKS-Storage-with-RDS-Database/README.md)
- Created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database.

### ALB Ingress Controller & External DNS

- Deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Installed **ALB Ingress Controller** as part of section [06-01-ALB-Ingress-Install](/06-ELB-Application-LoadBalancers/06-01-ALB-Ingress-Install/README.md)
- Installed **External DNS** as part of section [06-06-01-Deploy-ExternalDNS-on-EKS](/06-ELB-Application-LoadBalancers/06-06-ALB-Ingress-ExternalDNS/06-06-01-Deploy-ExternalDNS-on-EKS/README.md)

### XRay Daemon

- We are going to view the application traces in AWS X-Ray.
- We need XRay Daemon running as Daemonset for that.

```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns & xray-daemon pod running in default namespace
kubectl get pods
```

## Step-03: Review Deployment Manifest for V2 Notification Service

- We are going to distribute 50% traffic to each of the V1 and V2 version of application

| NS V1 Replicas | NS V2 Replicas | Traffic Distribution         |
| -------------- | -------------- | ---------------------------- |
| 2              | 2              | 50% traffic to NS V2 Version |

- **08-V2-NotificationMicroservice-Deployment.yml**

```yml
# Change-1: Image Tag is 4.0.0-AWS-XRay
    spec:
      containers:
        - name: notification-service
          image: stacksimplify/kube-notifications-microservice:4.0.0-AWS-XRay

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME
              value: "V2-Notification-Microservice"
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"
            - name: AWS_XRAY_CONTEXT_MISSING
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured
```

## Step-04: Review Ingress Manifest

- **07-ALB-Ingress-SSL-Redirect-ExternalDNS.yml**

```yml
# Change-1: Update with your SSL Cert ARN when using template
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:eu-west-1:180789647333:certificate/9f042b5d-86fd-4fad-96d0-c81c5abc71e1

# Change-2: Update with your "yourdomainname.com"
# External DNS - For creating a Record Set in Route53
external-dns.alpha.kubernetes.io/hostname: canarydemo.awseksstudy.com
```

## Step-05: Deploy Manifests

```
# Deploy
kubectl apply -f kube-manifests/

# Verify
kubectl get deploy,svc,pod
```

## Step-06: Test

```
# Test
https://canarydemo.awseksstudy.com/usermgmt/notification-xray

# Your Domain Name
https://<Replace-your-domain-name>/usermgmt/notification-xray
```

## Step-07: What is happening in the background?

- As far as `Notification Cluster IP Service` selector label matches to `Notificaiton V1 and V2 Deployment manifests selector.matchLables` those respective pods are picked to send traffic.

```yml
# Notification Cluster IP Service - Selector Label
  selector:
    app: notification-restapp

# Notification V1 and V2 Deployment - Selector Match Labels
  selector:
    matchLabels:
      app: notification-restapp
```

## Step-08: Clean-Up

- We are going to delete applications created as part of this section

```
# Delete Apps
kubectl delete -f kube-manifests/
```

## Step-09: Downside of this approach

- To change the percentage of traffic to Prod and Canary pods, we need to play with ReplicaSet - replicas.

## Step-10: Best ways for Canary Deployments

- Istio (Open Source)
- AWS AppMesh (AWS version of Istio)
