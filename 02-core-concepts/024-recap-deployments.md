# Recap - Deployments

Introduction

- In production, you usually run many pods copies of a container
- When upgrading, you don't want to close all pods and replace them at the same time, you want rolling updates
- If an upgrade leads to errors, you want to be able to rollback changes

These are deployments

- Give us ability to pause/resume rollouts
- Gives us ability for rolling updates

The YAML file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
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

Create it:

```shell
$ kubectl create -f deployment-definition.yml
```

Get the deployments:

```shell
$ kubectl get deployments
```

Get replicasets created by the deployment:

```shell
$ kubectl get replicaset
```

Get the pods:

```shell
$ kubectl get pods
```

View all objects

```shell
$ kubectl get all
```
