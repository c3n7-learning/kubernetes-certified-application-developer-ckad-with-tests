# Taints and Tolerations vs Node Affinity

Taints keep unwanted pods off a node.

- However, taints and tolerations alone don't guarantee that a pod lands on a particular node.

`nodeAffinity` attracts pods to the right node.

- However, it doesn't stop other pods from being scheduled on that node.

Combine both to dedicate nodes to specific pods:

- Use taints and tolerations to keep other pods off the dedicated nodes.
- Use `nodeAffinity` to ensure your pods are scheduled on those nodes and nowhere else.
