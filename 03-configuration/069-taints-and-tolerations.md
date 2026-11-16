# Taints and Tolerations

## Introduction

How do we control which pods go to which nodes, we use Taints and Tolerations. Are used to set restrictions on what nodes can be scheduled on a node. **Taints** are applied to nodes to repel pods, while **tolerations** are applied to pods to allow them to schedule onto nodes with matching taints

- **Taints**: (on nodes) You place a taint on a node to mark it as restricted. Any pod without a matching toleration is blocked or evicted depending on the taint's effect.
- **Tolerations**: (on pods) You add a toleration to a pod's manifest so the scheduler knows the pod is permitted to run on a tainted node.

Think of a Taint as Bug spray on the nodes, and Tolerations as which pods are tolerant to the bug spray

## Taints - Node

To taint a node:

```shell
$ kubectl taint nodes node-name key=value:<taint-effect>
```

There are three taint effects

- **`NoSchedule`**
  - **Behavior for non-tolerating pods:** Hard block; new pods will not be scheduled onto the node.
  - **Impact on already running pods:** Existing pods remain running on the node.
- **`PreferNoSchedule`**
  - **Behavior for non-tolerating pods:** Soft block; the scheduler tries to avoid placing new pods on the node, but will if no other resources exist.
  - **Impact on already running pods:** Existing pods remain running on the node.
- **`NoExecute`**
  - **Behavior for non-tolerating pods:** Hard block; new pods will not be scheduled onto the node.
  - **Impact on already running pods:** Existing pods are evicted immediately unless they have a matching toleration (or after an optional `tolerationSeconds` timer expires).

Example:

```shell
$ kubectl taint nodes node1 app=myapp:NoSchedule
```

## Tolerations - PODs

Assume a node with the taint:

```shell
$ kubectl taint nodes node1 app=blue:NoSchedule
```

The pod should look like:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations: # All values in double quotes
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

> Note:  
> Taints and Tolerations don't force a pod to a particular node. It tells a node to only accept pods with certain tolerations.  
> To force a pod to go specific nodes, it is achieved with **Node Affinity**

We have been considering the Worker Nodes only

- What of the master node?
- Master Node has a taint that prevents pods from going to it

```shell
$ kubectl describe node kubemaster | grep Taint

Taints:             node-role.kubernetes.io/master:NoSchedule
```

TO remove a taint from a node, add the `-` suffix like:

```shell
$ kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-

node/controlplane untainted
```
