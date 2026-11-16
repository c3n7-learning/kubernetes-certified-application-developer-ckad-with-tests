# Multi-container Pods Designer Patterns

We just saw **co-located** pods:

- There is no particular order in which they are started

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080
    - name: main-app
      image: main-app
```

**Init containers**:

- The init-container does its job and ends its job, and then the main application starts
- This could be a container that waits for a database to be ready before starting the main app

**Sidecar containers**:

- The sidecar starts first, does it's job, but is not terminated
- It continues to run throughout the whole lifecycle of the app
- You could have a log-shipper running as a side-car

```
Co-located            Regular Init           Sidecar
Containers            Containers             Containers

┌────────┐            ┌────────┐
│  (A)   │            │   ✔    │
└───┬────┘            └───┬────┘             ┌────────┐
    │                     │                  │        │
┌───┴────┐            ┌───┴────┐             │  Main  ├──(s)
│  Main  │            │  Main  │             │        │
└────────┘            └────────┘             └────────┘

Run side by side      Init runs first,       Helper runs alongside
in the same Pod       completes, then        the main container
                      main starts            for its whole lifetime
```

## Co-Located Containers:

The YAML is:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080
    - name: main-app
      image: main-app
```

## Init Containers:

The YAML is:

- The init containers are run sequentially, and once they complete, the main container is started

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080
  initContainers:
    - name: db-checker
      image: busybox
      command: "wait-for-db-to-start.sh"
    - name: api-checker
      image: busybox
      command: "wait-for-another-api.sh"
```

## Sidecar Containers:

The YAML is:

- Similar to init containers, but difference is `restartPolicy: Always`
- The container is started before the main container, and is terminated after the main container
- For a log shipper, this allows it to capture startup and termination logs

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080
  initContainers:
    - name: log-shipper
      image: busybox
      command: "setup-log-shipper.sh"
      restartPolicy: Always
```
