# Readiness and Liveness Probes

## Lifecycle of a Pod

**Pod Status** tells the status of a pod in its lifecycle

- Before a pod is scheduled, it is in a `Pending` state
- Once a pod is scheduled, it is in a `ContainerCreating` status. The images required for the application are pulled, and the container starts.
- Once a container starts, it is in the `Running` status, which it will remain in until it is terminated.
- Can be seen via `kubectl describe pod` or `kubectl get pods`

**Pod Conditions** complement pod status. It is an array of true/false values that tell us the state of a pod. It has:

- `PodScheduled` (true/false): Is true when a pod is scheduled on a node
- `Initialized` (true/false): When a pod is initialized, a pod is set to true
- `ContainersReady` (true/false): Once all containers in a pod are ready, is set to true
- `Ready` (true/false): Once the pod is considered to be are ready, is set to true
- Can be seen via `kubectl describe pod`

## `Ready` condition

Means the containers in a pod are ready to accept user traffic

- Some services take milliseconds, or seconds to get ready, or warm-up
- During this warm-up period, you don't want users being directed towards the app

By default, k8s assumes that a container is ready to serve traffic as soon as the container is up

- What if the pod has warm-up steps?
- The user sees the warm-up error screens, which is wrong!
- We want to do this check inside k8s

## Readiness Probes

How to set them up?

- You can set up probes e.g. `HTTP Test - /api/ready`
- You can do a tcp test `3306`
- You can execute a custom command

The YAML for HTTP:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
```

The YAML for a TCP probe:

```yaml
readinessProbe:
  tcpSocket:
    port: 3306
```

The YAML for a command probe:

```yaml
readinessProbe:
  exec:
    command:
      - cat
      - /app/is_ready
```

### More Options

If you know your app takes 10 seconds to warm up:

```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10
```

If you want to specify how often to probe

```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

By default, k8s gives up if the probe failes 3 times. To tweak this behaviour:

```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 8
```

## Readiness probes in multi-pods set-ups

Assume you have a replica set with multiple pods, and a service routing traffic to the pods

- When you add a new pod, without readiness probes, the new pod starts new traffic even if its not ready
- With readiness probes, the service waits until the pod is ready before it begins to route traffic to it
