# Kubernetes Namespaces - LimitRange - Declarative using YAML

## Step-01: Create Namespace manifest

- **Important Note:** File name starts with `00-` so that when creating k8s objects namespace will get created first so it don't throw an error.

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: dev3
```

## Step-02: Create LimitRange manifest

- Instead of specifying `resources like cpu and memory` in every container spec of a pod defintion, we can provide the default CPU & Memory for all containers in a namespace using `LimitRange`

```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ns-resource-quota
  namespace: dev3
spec:
  limits:
    - default:
        memory: "512Mi" # If not specified the Container's memory limit is set to 512Mi, which is the default memory limit for the namespace.
        cpu: "500m" # If not specified default limit is 1 vCPU per container
      defaultRequest:
        memory: "256Mi" # If not specified default it will take from whatever specified in limits.default.memory
        cpu: "300m" # If not specified default it will take from whatever specified in limits.default.cpu
      type: Container
```

## References:

- https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/
