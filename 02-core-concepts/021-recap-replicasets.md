# Recap - ReplicaSets

Controllers monitor k8s objects, and respond accordingly. We'll look at the replication controllers for now

## High Availability

If we have one pod, and the one pod fails, what happens?

- We disappoint our users
- We want to have multiple instances of a single pod, thus providing high availability

Does that mean we can't use replication controllers if we have one pod?

- Absolutely not. We can use replication controllers even if we have one pod or many
- It will create a new instance if the current one fails

```text
+------------------------------------------------------+
|  Node                                                |
|  +------------------------------------------------+  |
|  |                                                |  |
|  |  +----------------+     +----------------+     |  |
|  |  |                |     |                |     |  |
|  |  |    [ Py ]      |     |    [ Py ]      |     |  |
|  |  |                |     |                |     |  |
|  |  |  (o) Pod       |     |  (o) Pod       |     |  |
|  |  +----------------+     +----------------+     |  |
|  |                                                |  |
|  |            Replication Controller              |  |
|  +------------------------------------------------+  |
|                                                      |
+------------------------------------------------------+
```

## Load Balancing and Scaling

Replication controllers help to create multiple instances of a pod, and share the load across them

- Replication controllers span across multiple nodes in the cluster
- Helps scale up the number of instances

## `ReplicationController` vs `ReplicaSet`

Replication Controller and ReplicaSet are not the same

- Replication Controller is being deprecated
- ReplicaSet is the latest and greatest
- Minor differences between the two, but they are more or less the same

### `ReplicationController`

The YAML looks like:

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end

spec:
  template:
    # Our Pod details move here
```

Remember our pod spec?

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp

spec:
  containers:
    - name: nginx-container
      image: nginx
```

Update `ReplicationController` to be:

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end

spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp

    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
```

To apply it:

```shell
$ kubectl crate -f rc-definition.yml
```

To get it:

```shell
$ kubectl get replicationcontroller
```

To see the pods created by the replication controller

```shell
$ kubectl get pods

NAME             READY   STATUS    RESTARTS   AGE
myapp-rc-4lvk9   1/1     Running   0          20s
myapp-rc-mc2mf   1/1     Running   0          20s
myapp-rc-px9pz   1/1     Running   0          20s
```

### `ReplicaSet`

The YAML looks like:

- Pretty much looks like a `ReplicationController`:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end

spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp

    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  # ...
```

- The difference is that we requre a `spec.selector` definition
- Helps the replica set identify which pods fall under it.
- Why not just rely on `spec.template`? Because a replica set can manage pods that are not part of its spec definition
- Replication Controller has `spec.selector` definition as well, but it's optional

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

Apply it:

```shell
$ kubectl create -f code/021/rs-definition.yml
```

Get it

```shell
$ kubectl get replicaset
```

Get the pods:

```shell
$ kubectl get pods
```

## Labels and Selectors

How does the replicaset know which controller it should manage?

- We could have many other pods in the cluster
- We use `selector.matchLabels` to tell the replicaset which pods it should monitor and manage

Remember this statement?

> Why not just rely on `spec.template`? Because a replica set can manage pods that are not part of its spec definition

- If we had 3 instances of `myapp-pod` already running before creating the replicaset, do we still need to specify `spec.template`?
- Yes. Because if any of the pod fails, the replicaset needs to know how to create a new one

## Scaling Replicasets

### Declaratively

Simply update `spec.replicas` then update the replicaset:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end

spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 6
  selector:
    matchLabels:
      type: front-end
```

Apply it:

```shell
$ kubectl create -f code/021/rs-definition.yml
```

Get it

```shell
$ kubectl get replicaset
```

Get the pods:

```shell
$ kubectl get pods
```

### Using the CLI

We can use the cli to update the replicaset

```shell
$ kubectl scale --replicas=6 <type> <name>
$ kubectl scale --replicas=6 replicaset myapp-replicaset
```

## Commands

Create a replica set

```shell
$ kubectl crate -f replicaset-definition.yml
```

View replicasets:

```shell
$ kubectl get replicaset
```

Delete replicaset (with its pods):

```shell
$ kubectl delete replicaset myapp-replicaset
```

Replace or update the replicaset:

```shell
$ kubectl replace -f replicaset-def
```

Scale the replicaset (without modifying the spec file):

```shell
$ kubectl scale -replicas=6 -f replicaset-definition.yml
```
