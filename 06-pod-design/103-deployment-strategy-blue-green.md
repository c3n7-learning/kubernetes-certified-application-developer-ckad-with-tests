# Deployment Strategy - Blue Green

We previously discussed two deployment strategies:

- `Recreate`: Terminates all existing pods before creating new ones. This causes brief downtime between versions, but it guarantees that only one version runs at a time. It's useful when versions can't coexist (e.g., incompatible schema changes or exclusive resource locks).
- `RollingUpdate` (default): Gradually replaces old pods with new ones, controlled by `maxUnavailable` and `maxSurge`. The application stays available during the rollout, and you can roll back if something goes wrong. Both versions run side by side briefly, so they must be compatible.

There are two others which aren't explicitly in k8s:

- `Blue/Green` deployments
- `Canary` deployments

## `Blue/Green` deployments

Both the old (blue) and new (green) versions are fully deployed side by side. Live traffic keeps going to blue while the green version is tested. Once the tests pass, traffic is switched to green. Blue can be kept around for a quick rollback.

- Kubernetes has no built-in Blue/Green strategy (only `Recreate` and `RollingUpdate`), but you can implement it natively by running two Deployments and switching a Service's selector from one to the other.
- Service meshes (e.g., Istio), ingress controllers, or tools like Argo Rollouts can also do it, and they give finer-grained traffic control. The main trade-off is that it temporarily requires double the resources.

To do it in k8s:

- Rollout the blue deployment with a selector such as `version: v1`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        version: v1
    spec:
      containers:
        - name: app-container
          image: myapp-image:1.0
  replicas: 5
  selector:
    matchLabels:
      version: v1
```

- Have the service selector at `version: v1`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    version: v1
```

- Rollout the green deployment with a selector such as `version: v2`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        version: v2
    spec:
      containers:
        - name: app-container
          image: myapp-image:2.0
  replicas: 5
  selector:
    matchLabels:
      version: v2
```

- Change the service selector to `version: v2`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    version: v2
```
