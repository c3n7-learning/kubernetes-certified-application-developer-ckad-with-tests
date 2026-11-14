# Services - ClusterIP

## Introduction

- A typical application runs multiple pods, e.g. **front-end**, **back-end** and **redis**, and these need to communicate.

**What's the right way to connect them?**

- Pod IPs aren't static. They change whenever pods are recycled or rescheduled.
- With multiple replicas of a pod, there's no way to know which one to call.

**ClusterIP Service**

- Gives a single, stable endpoint for a group of pods.
- Forwards each request to one of the pods in the group, picked at random.

## ClusterIP

A **ClusterIP Service** is the default Kubernetes Service type. It gives a group of pods a single, stable virtual IP address (and DNS name) that is reachable only from inside the cluster. Pods are selected using label selectors, and incoming requests to the Service are load-balanced across the matching healthy pods, so clients don't need to know individual pod IPs, which change as pods are created, deleted or rescheduled.

For example, a front-end can reach the back-end through a name like `backend-service` instead of tracking any specific pod.

The YAML file

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
  selector:
    app: myapp
    type: back-end
```

Create the service:

```shell
$ kubectl create -f svc-definition.yml
```

View the services:

```shell
$ kubectl get services

NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   16d
back-end     ClusterIP   10.106.127.123   <none>        80/TCP    2m
```

Now we can access it using the service name or cluster-ip
