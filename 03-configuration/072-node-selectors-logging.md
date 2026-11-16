# Node Selectors Logging

## Introduction

Assume you have three nodes

- Node 1: 4xl
- Node 2: 2xl
- Node 3: xl

Assume you want pods to go to the largest node, so that they have buffer of resources to use should they need more RAM/CPU. How to do this?

On the POD:

- The node should already have the label `size=Large`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  nodeSelector:
    size: Large
```

- First label the node:

```bash
$ kubectl label nodes <node-name> <label-key>=<label-value>
$ kubectl label nodes node-1 size=Large
```

- Then create the pod:

```bash
$ kubectl create -f pod.yaml
```

Now the node is placed on node-1

## Complex Requirements

How to use the following conditions?

- Place on `Large` or `Medium` nodes
- Place on a node that is `Not Small`

Node Affinity was introduced to resolve this
