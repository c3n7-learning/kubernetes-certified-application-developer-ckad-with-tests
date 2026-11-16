# Recap - Namespaces

Introduction

- So far, we've created pods, replicasets, deployments
- They've been created in the `default` namespace
- k8s creates this for us by default
- In addition, k8s created `kube-system` and `kube-public` namespaces as well

Why namespaces?

- If starting up working with a small nbr of resources, you can use the `default` namespace
- You could, however, use different namespaces for different _environments_
- You could impose restrictions (access, auth, etc) on a namespace

How about connection?

- Services within a namespace can access each other over, say `db-service`
- Services outside the current namespace can be accessed via `<service-name>.<namespace>.svc.cluster.local`
  - e.g. `db-service.dev.svc.cluster.local`

Kubernetes Service DNS Name Breakdown

```python
mysql.connect("db-service.dev.svc.cluster.local")
```

The hostname follows the Kubernetes internal DNS format:

| Segment         | Label            | Description                                              |
| --------------- | ---------------- | -------------------------------------------------------- |
| `db-service`    | **Service Name** | The name of the Kubernetes Service you are connecting to |
| `dev`           | **Namespace**    | The namespace the Service lives in                       |
| `svc`           | **Service**      | Fixed subdomain indicating the record is for a Service   |
| `cluster.local` | **Domain**       | The cluster's default domain                             |

Format

```
<service-name>.<namespace>.svc.<cluster-domain>
```

Notes

- Within the same namespace, you can use just the short name (`db-service`).
- From a different namespace, use `db-service.dev` or the full name.

## Commands

Getting pods in the `default` namespace:

```bash
$ kubectl get pods
```

Getting pods in a different namespace:

```bash
$ kubectl get pods --namespace=kube-system
```

Ceating pods in the `default` namespace:

```bash
$ kubectl create -f pod-definition.yml
```

Ceating pods in the `dev` namespace (imperatively):

```bash
$ kubectl create -f pod-definition.yml --namespace=dev
```

Ceating pods in the `dev` namespace (declaratively):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev
  labels:
    app: myapp

spec:
  containers:
    - name: nginx-container
      image: nginx
```

## Create Namespace

Declaratively:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Create It:

```bash
$ kubectl create -f namespace-dev.yml
```

Imperatively:

```bash
$ kubectl create namespace dev
```

## Switching

Assume we have 3 namespaces

- `default`
- `dev`
- `prod`

If we run any kubectl command, it will run on the default namespace. What if we want to change the current namespace to something different?

- We will revisit this later

```bash
$ kubectl config set-context $(kubectl config current-context) --namespace=dev
```

To view resources in all namespaces, use the `--all-namespaces` option

```bash
$ kubectl get pod --all-namespaces
```

## Resource Quota

Declaratively

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests:memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

Create it:

```bash
$ kubectl create -f compute-quo
```
