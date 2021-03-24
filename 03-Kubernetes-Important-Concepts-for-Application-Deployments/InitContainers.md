# Kubernetes - Init Containers

## Step-01: Introduction

- Init Containers run **before** App containers
- Init containers can contain **utilities or setup scripts** not present in an app image.
- We can have and run **multiple Init Containers** before App Container.
- Init containers are exactly like regular containers, **except:**
  - Init containers always **run to completion.**
  - Each init container must **complete successfully** before the next one starts.
- If a Pod's init container fails, Kubernetes repeatedly restarts the Pod until the init container succeeds.
- However, if the Pod has a `restartPolicy of Never`, Kubernetes does not restart the Pod.

## Step-02: Implement Init Containers

- Update `initContainers` section under Pod Template Spec which is `spec.template.spec` in a Deployment

```yml
template:
  metadata:
    labels:
      app: usermgmt-restapp
  spec:
    initContainers:
      - name: init-db
        image: busybox:1.31
        command:
          [
            "sh",
            "-c",
            'echo -e "Checking for the availability of MySQL Server deployment"; while ! nc -z mysql 3306; do sleep 1; printf "-"; done; echo -e "  >> MySQL DB Server has started";',
          ]
```
