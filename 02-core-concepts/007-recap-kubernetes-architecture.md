# Recap - Kubernetes Architecture

## Introduction

Node

- Is a machine, physical or virtual, on which k8s is installed
- A worker node is a machine where containers will be launched
- Previously called 'minions'
- Ideally you need more than one for redundancy and high availability

Cluster

- Is a group of nodes

Master

- The master node is another node with k8s installed in it, and is configured as a master
- Watches all other nodes on the cluster
- Is responsible for orchestration of containers on the worker nodes

## Components

When you install k8s on a server, you install the following components

- **API Server**: is the 'frontend' of the cluster. Users, CLIs, etc talk to this to interface with the cluster
- **etcd**: is a distrubuted key-value store that stores all data necessary to manage the cluster.
- **Scheduler**: is responsible for distributing containers across multiple nodes
- **Controller**: are the brain behind orchestration. notice when containers are down, and make decisions on whether to restart them
- **Container Runtime**: is the underlying software that runs the containers. could be `docker` or `containerd`
- **Kubelet**: is the agent that runs on each node on the cluster. Is responsible for ensure the containers are running on the nodes as expected

## Master or Worker Nodes

Worker Nodes

- These have the **Container Runtime** running on them (docker or containerd, etc)
- These have `kubelet` installed on them that is responsible for providing health information of the worker nodes and perform actions requested by the master node

Master Nodes

- These have the `kube-apiserver` and this is what makes them the master
- The `etcd` lives here
- The `controller` lives here
- The `scheduler` is here as well

## Kubectl

Kube Command Line Tool or Kube Control

- Is used to deploy stuff on the cluster

Run a workload on the cluster:

```bash
kubectl run hello-minikube
```

Get info about the cluster

```bash
kubectl cluster-info
```

Get nodes in the cluster

```bash
kubectl get nodes
```
