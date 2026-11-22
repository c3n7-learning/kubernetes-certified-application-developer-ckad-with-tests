# Updating a Deployment

Here are some handy examples related to updating a Kubernetes Deployment.

## Creating a deployment, checking the rollout status and history

In the example below, we first create a simple deployment and inspect the rollout status and the rollout history:

```console
master $ kubectl create deployment nginx --image=nginx:1.16
deployment.apps/nginx created

master $ kubectl rollout status deployment nginx
Waiting for deployment "nginx" rollout to finish: 0 of 1 updated replicas are available...
deployment "nginx" successfully rolled out

master $ kubectl rollout history deployment nginx
deployment.extensions/nginx
REVISION  CHANGE-CAUSE
1         <none>

master $
```

## Using the `--revision` flag

Here revision 1 is the first version, where the deployment was created.

You can check the details of each revision individually by using the `--revision` flag:

```console
master $ kubectl rollout history deployment nginx --revision=1
deployment.extensions/nginx with revision #1

Pod Template:
 Labels:    app=nginx
            pod-template-hash=6454457cdb
 Containers:
  nginx:
   Image:       nginx:1.16
   Port:        <none>
   Host Port:   <none>
   Environment: <none>
   Mounts:      <none>
 Volumes:       <none>
master $
```

## Using the `--record` flag

You may have noticed that the `CHANGE-CAUSE` field is empty in the rollout history output. We can use the `--record` flag to save the command used to create/update a deployment against the revision number.

```console
master $ kubectl set image deployment nginx nginx=nginx:1.17 --record
deployment.extensions/nginx image updated

master $ kubectl rollout history deployment nginx
deployment.extensions/nginx

REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx nginx=nginx:1.17 --record=true
master $
```

You can now see that the change-cause is recorded for revision 2 of this deployment.

Let's make some more changes. In the example below, we edit the deployment and change the image from `nginx:1.17` to `nginx:latest`, again using the `--record` flag.

```console
master $ kubectl edit deployments. nginx --record
deployment.extensions/nginx edited

master $ kubectl rollout history deployment nginx
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx nginx=nginx:1.17 --record=true
3         kubectl edit deployments. nginx --record=true

master $ kubectl rollout history deployment nginx --revision=3
deployment.extensions/nginx with revision #3

Pod Template:
 Labels:       app=nginx
               pod-template-hash=df6487dc
 Annotations:  kubernetes.io/change-cause: kubectl edit deployments. nginx --record=true

 Containers:
  nginx:
   Image:       nginx:latest
   Port:        <none>
   Host Port:   <none>
   Environment: <none>
   Mounts:      <none>
 Volumes:       <none>

master $
```

## Undo a change

Let's now roll back to the previous revision. Running `kubectl rollout undo` without any flags rolls back to the revision immediately before the current one (here, revision 2 with `nginx:1.17`):

```console
controlplane $ kubectl rollout undo deployment nginx
deployment.apps/nginx rolled back
```

> **Note:** Kubernetes doesn't keep the old revision number. It re-applies the old pod template as a **new** revision at the top of the list (revision 4) and removes the old entry (revision 2). This is why the history below shows revisions 1, 3 and 4.

```console
controlplane $ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
3         kubectl edit deployments.apps nginx --record=true
4         kubectl set image deployment nginx nginx=nginx:1.17 --record=true

controlplane $ kubectl rollout history deployment nginx --revision=3
deployment.apps/nginx with revision #3
Pod Template:
  Labels:       app=nginx
                pod-template-hash=787f54657b
  Annotations:  kubernetes.io/change-cause: kubectl edit deployments.apps nginx --record=true
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>

controlplane $ kubectl describe deployments. nginx | grep -i image:
    Image:        nginx:1.17

controlplane $
```

With this, we have rolled back to the previous version of the deployment, with the image `nginx:1.17`.

To roll back to a specific revision, use the `--to-revision` flag. First, inspect revision 1:

```console
controlplane $ kubectl rollout history deployment nginx --revision=1
deployment.apps/nginx with revision #1
Pod Template:
  Labels:       app=nginx
                pod-template-hash=78449c65d4
  Containers:
   nginx:
    Image:        nginx:1.16
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>

controlplane $ kubectl rollout undo deployment nginx --to-revision=1
deployment.apps/nginx rolled back
```

With `--to-revision=1`, the deployment is rolled back to the first image we used to create it, as we can see in the rollout history output:

```console
controlplane $ kubectl describe deployments. nginx | grep -i image:
    Image:        nginx:1.16
```
