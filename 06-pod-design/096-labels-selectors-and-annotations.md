# Labels, Selectors and Annotations

Labels are a way to group things

- Allow us to also filter out things as a group
- **Labels** are properties attached to items
- **Selectors** are a way to filter these items

In kubernetes:

- All objects in k8s can have labels attached

## Labels and selectors in k8s

Create the pod with `metadata.labels`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1
    Function: Front-end
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
```

To select pods with a label

```bash
$ kubectl get pods --selector app=App1
```

For ReplicaSets:

- Note that the labels for the replicaset (`metadata.labels`) are for the replicaset alone, and the labels for the pod (`spec.template.metadata.labels`) are for the pod alone (no correlation)
- Meaning, on `spec.matchLabels`, you need at least one label to match the labels in `spec.template.metadata.labels`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

For services:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: App1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

## Annotations

Annotations are used to record other details for informatory purposes

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
  annotations:
    buildversion: "1.34"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

The highlighted `annotations` block sits under `metadata`, alongside `labels`. Unlike labels, annotations aren't used for selecting objects; they just store extra info like build version or contact details.
