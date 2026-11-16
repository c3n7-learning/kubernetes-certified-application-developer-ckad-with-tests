# Resource Requirements

The k8s `kube-scheduler` makes decisions on where to place a node.

- If the resources needed for a pod are not available on node 1, it will be sheduled on node 2 (if it has the needed resources)
- If no node has the required resources, the pod is stuck in the pending state

## Resource Requests

Assume a container needs 2CPU and 4Gi Mem, how do you specify that?

- The `kube-scheduler` will look for a node that guarantees atleast this.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "4Gi"
          cpu: 2
```

### Resource - CPU

What does 1 CPU mean? (or 1000m - milli-cpus)

- 1 AWS vCPU
- 1 GCP Core
- 1 Azure Core
- 1 Hyperthread

### Resource - Memory

You can specify in bytes or G, M, K, Gi, Mi, Ki, etc

- **1 G** (Gigabyte) = 1,000,000,000 bytes
- **1 M** (Megabyte) = 1,000,000 bytes
- **1 K** (Kilobyte) = 1,000 bytes
- **1 Gi** (Gibibyte) = 1,073,741,824 bytes
- **1 Mi** (Mebibyte) = 1,048,576 bytes
- **1 Ki** (Kibibyte) = 1,024 bytes

Example:

- 1024 or 1Ki are both valid

## Resource Limits

Assume we specified the `request` limit to be 2 vCPU and 4Gi RAM

- The container by default can use as much CPU or as much RAM as possible
- We can control this by specifying the `spec.containers.resources.limits`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "1Gi"
          cpu: 2
        limits:
          memory: "2Gi"
          cpu: 2
```

- If a container exceeds its **CPU limit**, it is **throttled** (slowed down, not killed).
- If it exceeds its **memory limit**, it is terminated with an **Out of Memory (OOM)** error, shown as `OOMKilled` with exit code 137, and restarted per the Pod's `restartPolicy`.
- Separately, if the **node** runs low on memory, Pods can be **evicted**.

## Default Behavior

**No requests, no limits**

- A pod can use as much CPU/RAM as the node allows (`BestEffort`, first to be evicted)
- The pod is not guaranteed anything (QoS class `BestEffort`), and it is the first to be evicted when a node is under pressure. It is still bounded by the node's real capacity, and a namespace `LimitRange` can inject default values.

**No requests, with limits**

- Requests are automatically set equal to limits
- The pod is only scheduled on nodes that can provide the amount in `spec.containers.resources.limits`

**Requests and limits**

- The pod is scheduled on nodes that satisfy the requests
- CPU above the limit is throttled; memory above the limit is OOM-killed

**Requests, no limits**

- The pod is scheduled on nodes that satisfy the requests
- It can use more resources if needed, up to what the node has free

## `LimitRange`

### CPU Limits

Can help you set the limit ranges for all pods, at the namespace level:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resouce-constraint
spec:
  limits:
    - default: # Limit
        cpu: 500m
      defaultRequest: # Request
        cpu: 500m
      max: # Limit
        cpu: "1"
      min: # Request
        cpu: 100m
      type: Container
```

**Why `default` and `max` are different even though both relate to limits.** `default` is what you get if you say nothing. `max` is the ceiling even if you do say something. You could set `default: 500m` and `max: 1`, so developers get a modest default but can raise it up to 1 CPU if they need to.

- **`default` and `defaultRequest`** fill in values when you don't specify any.
- **`min` and `max`** reject Pods whose values fall outside the range.

| Field            | Job                 | What it does                                              |
| ---------------- | ------------------- | --------------------------------------------------------- |
| `default`        | Default **limit**   | If a container has no CPU limit, this value is injected   |
| `defaultRequest` | Default **request** | If a container has no CPU request, this value is injected |
| `max`            | Upper bound         | A container whose **limit** is above this is rejected     |
| `min`            | Lower bound         | A container whose **request** is below this is rejected   |

**Why you need both kinds.** The defaults never reject anything, they only supply missing values. The min/max never supply anything, they only validate. With your example:

- A container with no resources set gets a request of `500m` and a limit of `500m`. Both are within the range, so it is admitted.
- A container that asks for `cpu: 50m` is **rejected**, because it's below `min: 100m`.
- A container that sets a limit of `2` is **rejected**, because it's above `max: "1"`.
- A container that sets a limit of `800m` is fine. The defaults don't apply because it specified its own value, and `800m` is within the range.

**Gotcha:** the defaults must fall inside the min/max range, otherwise the LimitRange is invalid. Also, a LimitRange only affects Pods created **after** it exists, and it applies per namespace.

TLDR;

- **`default`:** limit injected when a container doesn't specify one.
- **`defaultRequest`:** request injected when a container doesn't specify one.
- **`max`:** highest limit allowed; Pods above it are rejected.
- **`min`:** lowest request allowed; Pods below it are rejected.
- Defaults supply missing values; min/max validate values. Defaults must lie within min and max.
- Applies per namespace, and only to Pods created after the LimitRange.

### RAM Limits

It looks more or less the same:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
    - default:
        memory: 1Gi
      defaultRequest:
        memory: 1Gi
      max:
        memory: 1Gi
      min:
        memory: 500Mi
      type: Container
```

## Resource Quotas

A **ResourceQuota** is a namespace-level object that caps the total resources (CPU, memory, etc.) that all Pods in that namespace can request or use combined.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```
