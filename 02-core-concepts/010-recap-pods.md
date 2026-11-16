# Recap - Pods

## Pre-Requisites

- The docker image you want to use is on docker hub
- The k8s cluster is already set-up
- All services need to be in a running state

## Introduction

Containers don't run directly, they need to be encapsulated in a **Pod**:

- A pod is a single instance of an application
- Is the simplest and smallest workload that you can run on k8s

What if we receive traffic? what do we do?

- ❌ We do NOT provision another container on the same pod
- ✅ Instead, we provision more pods on the node!

What if we fill up the node?

- We add more nodes!

## Multi-Container Pods

We just said that pods usually have a one-to-one relationship with containers

- A pod can have multiple containers
- Usually though, the containers are not of the same kind
- Sometimes, you can need a helper container to do some-supporting tasks
  - e.g. processing files, etc etc
- When the **application container** is created, the **helper container** is created, when the container dies, the helper container dies, since they are part of the same pod
- They can reference each other over `localhost` since they share the same network

```ascii
┌────────────────────────────────────────────────────────┐
│ NODE                                                   │
│                                                        │
│    ┌──────────────────────────────────┐                │
│    │ POD                              │                │
│    │                                  │                │
│    │  ┌────────────┐  ┌────────────┐  │   ╭──────────╮ │
│    │  │|| Python ||│  │||  .NET  ||│  │   │ Helper   │ │
│    │  │|| ||  || ||│  │|| ||  || ||│  │◄──│Containers│ │
│    │  └─────┬──────┘  └──────┬─────┘  │   ╰──────────╯ │
│    │        │   ( Network )  │        │                │
│    │        └────────────────┘        │                │
│    └──────────────────────────────────┘                │
│                                                        │
└────────────────────────────────────────────────────────┘
```

## Pods Again!

Let's approach it a different way

Scaling in docker

- You simply run more containers

```bash
$ docker run python-app
$ docker run python-app
$ docker run python-app
$ docker run python-app
```

We now have a helper container that helps our web app by processing or fetching data from elsewhere

- They maintain a one-to-one link with the application container
- We need to maintain :
  - a map of what connects to what using links and custom networks, volumes,
  - state monitoring to kill the helper container if the application container is killed

```bash
$ docker run helper -link app1
$ docker run helper -link app2
$ docker run helper -link app3
$ docker run helper -link app4
```

| App     | Helper | Volume |
| ------- | ------ | ------ |
| Python1 | App1   | Vol1   |
| Python2 | App2   | Vol2   |

- k8s does all this for us, for free!

## Pods in k8s

We previously saw:

```bash
$ kubectl run nginx
```

It:

- Creates a pod automatically and deploys an instance of the nginx docker image

Where does it get the image from?

- By specifying the source with `--image`

```bash
# kubectl run nginx --image nginx
```

Getting pods in our cluster:

```bash
$ kubectl get pods

NAME                    READY   STATUS              RESTARTS   AGE
nginx-8586cf59-whssr    0/1     ContainerCreating   0          3s

nginx-8586cf59-whssr    1/1     Running             0          8s
```
