# Canary Deployment

## Introduction

- Deploy the new version alongside the old one, but route only a small percentage of traffic to it (the "canary").
- Monitor the canary (errors, latency, metrics). If it looks healthy, gradually increase its share of traffic, or update the main Deployment to the new version and remove the canary.
- If it misbehaves, remove the canary. Only a small fraction of users were affected.
- Once the canary passes testing, update the primary Deployment to the new version (e.g., change its image), then delete the canary Deployment so all traffic runs on the new version.
- In plain Kubernetes, the traffic split depends on pod counts, which limits precision. For exact percentages, use a service mesh like Istio.

## How do you route one Service to two Deployments (primary and canary)?

- Give both Deployments' pods a common label (e.g., `app: front-end`) and use it as the Service's selector.
- By default, the Service spreads traffic across all matching pods, so the split follows the pod count, not the Deployment count. With equal replicas, you get roughly 50%/50%.
- To shrink the canary's share, reduce its replicas. For example, 3 primary pods and 1 canary pod means about 25% of traffic goes to the canary.

## How to go about it

Create the primary deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-primary
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        version: v1
        app: front-end
    spec:
      containers:
        - name: app-container
          image: myapp-image:1.0
  replicas: 5
  selector:
    matchLabels:
      app: front-end
```

The canary deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        version: v2
        app: front-end
    spec:
      containers:
        - name: app-container
          image: myapp-image:2.0
  replicas: 1
  selector:
    matchLabels:
      app: front-end
```

The service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: front-end
```
