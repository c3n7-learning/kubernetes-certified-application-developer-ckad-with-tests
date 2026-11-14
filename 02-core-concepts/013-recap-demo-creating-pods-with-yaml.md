# Recap - Demo - Creating Pods with YAML

## Pre-Requisites

Run a k3d cluster: https://k3d.io/stable/usage/exposing_services/#2-via-nodeport

```shell
$ k3d cluster create mycluster -p "8082:30080@agent:0" --agents 2
```

1. Create [`pod-definition.yaml`](code/013/pod-definition.yaml)
2. Apply it: `kubectl create -f code/013/pod-definition.yaml`

```
kubectl get pods -w
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   0/1     Pending   0          0s
myapp-pod   0/1     Pending   0          0s
myapp-pod   0/1     ContainerCreating   0          0s
myapp-pod   1/1     Running             0          21s
```

> Note:  
> In the exam only `vi` or `nano` are available. no GUI editing tool is available

A trick for creating a YAML file is

```shell
$ kubectl run redis --image=redis123 --dry-run=client -o yaml
$ kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
```
