# Kubernetes - Liveness & Readiness Probes

## Step-01: Create Liveness Probe with Command

```yml
livenessProbe:
  exec:
    command:
      - /bin/sh
      - -c
      - nc -z localhost 8095
  initialDelaySeconds: 60
  periodSeconds: 10
```

## Step-02: Create Readiness Probe with HTTP GET

```yml
readinessProbe:
  httpGet:
    path: /usermgmt/health-status
    port: 8095
  initialDelaySeconds: 60
  periodSeconds: 10
```

- **Observation:** User Management Microservice pod will not be in READY state to accept traffic until it completes the `initialDelaySeconds=60seconds`.
