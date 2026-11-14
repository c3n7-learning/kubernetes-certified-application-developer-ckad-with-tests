# Recap - Pods with YAML

## Introduction

A k8s YAML definition file always contains the following **root-level properties**

```yaml
apiVersion:
kind:
metadata:

spec:
```

`apiVersion` (string):

- Is the version of the k8s API we are using to create the object
- Could be:
  - Pod: `v1`
  - Service: `v1`
  - ReplicaSet: `apps/v1`
  - Deployment: `apps/v1`

`kind` (string):

- The type of object we're trying to create
- Could be `Pod`, `Service`, `ReplicaSet`, `Deployment`, etc

`metadata` (dictionary):

- Contains data that uniquely identifies the object within the cluster
- `labels` can contain any key->value pair of your choosing

`spec` (dictionary)

- Where we provide additional information to k8s depending on the resource we are trying to create
- For `Pod`, `spec.containers` is a list of dictionaries

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```

To apply it

```shell
$ kubectl create -f pod-definition.yaml
```

## Commands

To see pods in our cluster:

```shell
# kubectl get pods
```

To get detailed information about a pod:

```shell
$ kubectl describe pod myapp
```
