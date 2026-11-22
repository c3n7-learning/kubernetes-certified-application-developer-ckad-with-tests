# Rolling Updates & Rollbacks in Deployments

When you first create a deployment

- A new rollout is created
- A new rollout creates a new replicaset, which is recorded as a new deployment version (Revision 1)
- Another new rollout creates a new replicaset, which is recorded as a new deployment version (Revision 2)
- This allows us to roll back to a previous version should we need to

```
+- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -+
:                                                                          :
:              ||||    ||||    ||||    ||||    ||||    ||||    ||||    ||||:
:  Revision 1  |py|    |py|    |py|    |py|    |py|    |py|    |py|    |py|:
:              ||||    ||||    ||||    ||||    ||||    ||||    ||||    ||||:
:            nginx:   nginx:  nginx:  nginx:  nginx:  nginx:  nginx:  nginx:
:             1.7.0    1.7.0   1.7.0   1.7.0   1.7.0   1.7.0   1.7.0   1.7.0
:                                                                          :
+- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -+

+- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -+
:                                                                          :
:              ||||    ||||    ||||    ||||    ||||    ||||    ||||    ||||:
:  Revision 2  |py|    |py|    |py|    |py|    |py|    |py|    |py|    |py|:
:              ||||    ||||    ||||    ||||    ||||    ||||    ||||    ||||:
:            nginx:   nginx:  nginx:  nginx:  nginx:  nginx:  nginx:  nginx:
:             1.7.1    1.7.1   1.7.1   1.7.1   1.7.1   1.7.1   1.7.1   1.7.1
:                                                                          :
+- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -+
```

Checking the rollout status

- Run

```bash
$ kubectl rollout status deployment/myapp-deployment
```

- Output

```text
Waiting for rollout to finish: 0 of 10 updated replicas are available...
Waiting for rollout to finish: 1 of 10 updated replicas are available...
Waiting for rollout to finish: 2 of 10 updated replicas are available...
Waiting for rollout to finish: 3 of 10 updated replicas are available...
Waiting for rollout to finish: 4 of 10 updated replicas are available...
Waiting for rollout to finish: 5 of 10 updated replicas are available...
Waiting for rollout to finish: 6 of 10 updated replicas are available...
Waiting for rollout to finish: 7 of 10 updated replicas are available...
Waiting for rollout to finish: 8 of 10 updated replicas are available...
Waiting for rollout to finish: 9 of 10 updated replicas are available...
deployment "myapp-deployment" successfully rolled out
```

To view the rollout history/revision history:

- Run:

```bash
$ kubectl rollout history deployment/myapp-deployment
```

- Output:

```text
deployments "myapp-deployment"
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl apply --filename=deployment-definition.yml --record=true
```

## Deployment Strategy

How do we apply new pods?

- `Recreate`: We could kill all old pods, then rollout new pods
- `RollingUpdate` (default): We take down the older version, and bring up the newer version, pod by pod

You can look at the strategy via:

```bash
$ kubectl describe deployment my-deployment
```

## How to update a deployment

### Declaratively

Update your YAML:

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
          # from nginx:1.7.0
          image: nginx:1.7.1
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

Then apply it

```bash
$ kubectl apply -f deployment-definition.yaml
```

### Imperatively

Run:

```bash
$ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
```

## Upgrades

When you update a deployment

- A new replicaset is created
- The old replicaset is taken down using the Rolling Update strategy

The old replicaset is maintained though

```bash
$ kubectl get replicasets
```

To rollback:

- The new replicaset will be brought down, and the old our brought back up

```bash
$ kubectl rollout undo deployment myapp-deployment
```

## Commands Summary

Create

```bash
kubectl create -f deployment-definition.yml
```

Get

```bash
kubectl get deployments
```

Update

```bash
kubectl apply -f deployment-definition.yml
```

```bash
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
```

Status

```bash
kubectl rollout status deployment/myapp-deployment
```

```bash
kubectl rollout history deployment/myapp-deployment
```

Rollback

```bash
kubectl rollout undo deployment/myapp-deployment
```
