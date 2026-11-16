# Node Affinity

How to use the following conditions?

- Place on `Large` or `Medium` nodes
- Place on a node that is `Not Small`

Answer: Node Affinity

- This node selector:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  nodeSelector:
    size: Large
```

- Is equivalent to this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution::
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
```

This is quite verbose, but it allows us to do stuff like (`In` operator):

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution::
    nodeSelectorTerms:
      - matchExpressions:
          - key: size
            operator: In
            values:
              - Large
              - Medium
```

Or: with the `NotIn` operator

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution::
    nodeSelectorTerms:
      - matchExpressions:
          - key: size
            operator: NotIn
            values:
              - Small
```

With the `Exists` operator:

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution::
    nodeSelectorTerms:
      - matchExpressions:
          - key: size
            operator: Exists
```

## Operators

The following are all the logical operators that you can use in the `operator` field for `nodeAffinity` and `podAffinity`

- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#operators

| Operator       | Behavior                                                        |
| -------------- | --------------------------------------------------------------- |
| `In`           | The label value is present in the supplied set of strings       |
| `NotIn`        | The label value is not contained in the supplied set of strings |
| `Exists`       | A label with this key exists on the object                      |
| `DoesNotExist` | No label with this key exists on the object                     |

The following operators can only be used with `nodeAffinity`.

| Operator | Behavior                                                                                                                                                         |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Gt`     | The field value will be parsed as an integer, and the integer that results from parsing the value of a label named by this selector is greater than this integer |
| `Lt`     | The field value will be parsed as an integer, and the integer that results from parsing the value of a label named by this selector is less than this integer    |

## Node Affinity

What if no nodes are matched

- This is answered by the long property `requiredDuringSchedulingIgnoredDuringExecution`

Node Affinity Types:

- `requiredDuringSchedulingIgnoredDuringExecution`: The scheduler can't schedule the Pod unless the rule is met. This functions like `nodeSelector`, but with a more expressive syntax.
- `preferredDuringSchedulingIgnoredDuringExecution`: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

Pods already scheduled before the affinity is effected will remain unaffected
