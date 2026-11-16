# Kubectl Explain Command

If you forget resource names or shortnames or versions:

```bash
$ kubectl api-resources
```

To check details about a pod

```bash
$ kubectl explain pods
```

To go deeper

```bash
$ kubectl explain pods.spec
```

To show everything

```bash
$ kubectl explain pods --recursive
```
