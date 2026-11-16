# Commands and Arguments in Kubernetes

We previously created a custom docker container:

```shell
$ docker run --name ubuntu-sleeper ubuntu-sleeper
$ docker run --name ubuntu-sleeper ubuntu-sleeper 10
```

To specify the args in a k8s pod:

- To override the default value of 5 (`CMD ["5]`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"]
```

- To override the entry point (`ENTRYPOINT ["sleep"]`), use the `command` field

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args: ["10"]
```
