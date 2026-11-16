# Monitor and Debug Applications

## Monitoring

What to monitor on Kubernetes:

- Cluster level: number of nodes (and how many are healthy), node CPU/memory/disk
- Pod level: number of pods, pod status/restarts, CPU/memory usage

Common solutions:

- Metrics Server
- Prometheus
- Elastic Stack
- Datadog
- Dynatrace

## Metrics Server

- The in-cluster, lightweight successor to the deprecated **Heapster**
- Collects CPU and memory metrics from nodes and pods and keeps them **in memory only**, so there is **no historical data**. Use Prometheus, Elastic, etc. for history.

How metrics are collected:

- The **kubelet** runs on every node
- It includes **cAdvisor** (Container Advisor), which collects performance metrics from containers
- The kubelet exposes these metrics through its API, and Metrics Server scrapes them from there

## Setup

Minikube:

```bash
minikube addons enable metrics-server
```

Other environments (the old `deploy/1.8+/` path is outdated):

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

View metrics:

```bash
$ kubectl top node
NAME                     CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
k3d-mycluster-agent-0    26m          0%       183Mi           1%
k3d-mycluster-agent-1    28m          0%       205Mi           1%
k3d-mycluster-server-0   155m         1%       800Mi           6%
```

```bash
$ kubectl top pod

NAME             CPU(cores)   MEMORY(bytes)
ubuntu-sleeper   0m           1Mi
```
