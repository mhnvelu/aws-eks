# Kubernetes Namespaces - Imperative using kubectl

## Step-01: Introduction

- Namespaces allow to split-up resources into different groups.
- Resource names should be unique in a namespace
- We can use namespaces to create multiple environments like dev, staging and production etc
- Kubernetes will always list the resources from `default namespace` unless we provide exclusively from which namespace we need information from.

## Step-02: Namespaces Generic - Deploy in Dev1 and Dev2

### Create Namespace

```
# List Namespaces
kubectl get ns

# Create Namespace Imperatively
kubectl create namespace <namespace-name>
kubectl create namespace dev1
kubectl create namespace dev2

# List Namespaces
kubectl get ns
```

### Comment NodePort in UserMgmt NodePort Service

- **File: 08-UserManagement-Service.yml**
- **Why?:**
  - Whenever we create with same manifests multiple environments like dev1, dev2 with namespaces, we cannot have same worker node port for multiple services.
  - We will have port conflict.
  - Its good for k8s system to provide dynamic nodeport for us in such situations.

```yml
#nodePort: 31231
```

- **Error** if not commented

```log
The Service "usermgmt-restapp-service" is invalid: spec.ports[0].nodePort: Invalid value: 31231: provided port is already allocated
```

### Deploy All k8s Objects

```
# List all objects from dev1 & dev2 Namespaces
kubectl get all -n dev1
kubectl get all -n dev2
```

- **Shorter Note**
  - PVC is a namespace specific resource
  - PV and SC are generic
- **Observation-1:** `Persistent Volume Claim (PVC)` gets created in respective namespaces

- **Observation-2:** `Storage Class (SC) and Persistent Volume (PV)` gets created generic. No specifc namespace for them

## Step-05: Clean-Up

```
# Delete namespaces dev1 & dev2
kubectl delete ns dev1
kubectl delete ns dev2

# List all objects from dev1 & dev2 Namespaces
kubectl get all -n dev1
kubectl get all -n dev2

# List Namespaces
kubectl get ns

```

## References:

- https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/
