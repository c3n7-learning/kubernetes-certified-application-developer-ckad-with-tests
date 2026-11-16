# Security Contexts

In docker, we saw:

- To remove/add a capability to a docker container:

```shell
$ docker run --cap-add MAC_ADMIN ubuntu
$ docker run --cap-drop KILL ubuntu
```

- To run with all capabilities

```shell
$ docker run --privileged ubuntu
```

Kubernetes supports the same:

## Kubernetes Security

In k8s, containers are encapsulated in pods

- You can configure security on the pod level, or container level
- Settings on the container override any settings on the pod

Security context on the pod level:

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
```

Security context on the container level:

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```
