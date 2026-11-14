# Services

## Introduction

A Service gives pods a stable way to be reached, whether by other pods or by users outside the cluster.

- Connects applications to each other and to users
- Exposes pods to users
- Enables pod-to-pod communication
- Enables communication with external clients

## External Communication

- How do we access a pod from outside the cluster?
- A **NodePort** Service listens on a port on each node and forwards traffic to a port on a pod.

## Service Types

- **ClusterIP**: Exposes the Service on an internal virtual IP, so pods inside the cluster can reach it. This is the default type.
- **NodePort**: Exposes the Service on a static port on every node's IP, so external traffic can reach it at `<NodeIP>:<NodePort>`.
- **LoadBalancer**: Provisions an external load balancer in supported cloud providers and routes its traffic to the Service.

## Service - NodePort

NodePort has a valid range of `30_000` to `32_767`

```
+--------------------------------------------------+
| Node                                             |
|                                                  |
|                                                  |
| NodePort   +-----------+                         |
|  +-----+   |           |(Port)                   |
|  |30008|---|  Service  |80---+                   |
|  +-----+   |           |     |                   |
|            +-----------+     |                   |
|                              | (Target Port)     |
|                      +-------v-------+           |
|                      |      80       |           |
|                      |   10.244.0.2  |           |
|                      |    [Python]   |           |
|                      |      Pod      |           |
|                      +---------------+           |
|                                                  |
+--------------------------------------------------+
```

The YAML file looks like:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30080
  selector:
    app: myapp
    type: front-end
```

`spec.ports`: is an array:

- `port` is the port on the service object. (mandatory)
- `targetPort`: is the port on the pod. If unspecified, it's asssumed to be the same as `port`. (optional)
- `nodePort`: is any number in the valid range. (optional)

Create the service:

```shell
$ kubectl create -f svc-definition.yml
```

View the services:

```shell
$ kubectl get services
```

### Multiple Pods

If you have multiple pods

- If they have the same labels, and the labels are in the service's `spec.selector`, the nodeport maps to the pods
- The service then balances traffic across the multiple pods with the **Random** Algorithm

```
                   (User)
                     O
                    /|\
                    / \
                     :
                     :
+--------------------:--------------------------------------+
| 192.168.1.2        :                                      |
|              +-----------+                                |
|              |   30008   |   <- NodePort                  |
|              +-----------+                                |
|                    :                                      |
|              +-----------+                                |
|              |  Service  |                                |
|              +-----------+                                |
|                    :                                      |
|        +-----------+-----------+                          |
|        :           :           :                          |
|  +----------+ +----------+ +----------+                   |
|  |10.244.0.3| |10.244.0.2| |10.244.0.4|                   |
|  |          | |          | |          |                   |
|  | [Python] | | [Python] | | [Python] |                   |
|  |          | |          | |          |                   |
|  |   Pod    | |   Pod    | |   Pod    |                   |
|  +----------+ +----------+ +----------+                   |
|                                                           |
| Node                                                      |
+-----------------------------------------------------------+
```

What if the pods are on multiple nodes?

- k8s automatically creates a service that spans across all pods in the cluster

```
                         (User)
                           O
                          /|\
                          / \
                           |
        +------------------+------------------+
        :                  :                  :
+-------:---------+ +------:----------+ +-------:---------+
| 192.168.1.2     | | 192.168.1.3     | | 192.168.1.4     |
|    +------+     | |     +------+    | |        +------+ |
|    |30008 |     | |     |30008 |    | |        |30008 | |
|    +--:---+     | |     +--:---+    | |        +--:---+ |
|       :         | |        :        | |           :     |
| +-----------------------------------------------------+ |
+-|                       Service                       |-+
| +-----------------------------------------------------+ |
|       :         | |        :        | |           :     |
| +-----:-------+ | | +------:------+ | | +---------:---+ |
| | 10.244.0.3  | | | | 10.244.0.2  | | | | 10.244.0.4  | |
| |             | | | |             | | | |             | |
| |  [Python]   | | | |  [Python]   | | | |  [Python]   | |
| |             | | | |             | | | |             | |
| |    Pod      | | | |    Pod      | | | |    Pod      | |
| +-------------+ | | +-------------+ | | +-------------+ |
|                 | |                 | |                 |
| Node            | | Node            | | Node            |
+-----------------+ +-----------------+ +-----------------+
```

And you can access any of the pods by the <node-ip>:<targetPort>

```shell
$ curl http://192.168.1.2:30008
$ curl http://192.168.1.3:30008
$ curl http://192.168.1.4:30008
```
