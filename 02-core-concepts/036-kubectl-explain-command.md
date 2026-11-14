# Kubectl Explain Command

If you forget resource names or shortnames or versions:

```shell
$ kubectl api-resources
```

To check details about a pod

```shell
$ kubectl explain pods
```

To go deeper

```shell
$ kubectl explain pods.spec
```

To show everything

```shell
$ kubectl explain pods --recursive
```
