# Logging

How we see logs:

- In docker (with `-f` to follow logs)

```bash
$ docker logs my-container id
```

- In k8s

```bash
$ kubectl logs -f <my-pod-name>
```

- In k8s with a multi-container pod

```bash
$ kubectl logs -f <my-pod-name> -c <my-container-name>
# Or
$ kubectl logs -f <my-pod-name> <my-container-name>
```

This is all you need to know for the CKAD exam, but we'll see more.
