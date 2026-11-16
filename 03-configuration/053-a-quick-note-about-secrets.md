# A quick note about Secrets!

Remember that Secrets encode data in base64 format. Anyone with the base64-encoded Secret can easily decode it, so Secrets should not be considered very safe.

The safety of Secrets is a bit confusing in Kubernetes. The [Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/secret) and a lot of blogs refer to Secrets as a "safer option" for storing sensitive data. They are safer than storing data in plain text, as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion, it's not the Secret itself that is safe, it is the practices around it.

Secrets are not encrypted, so they are not safer in that sense. However, some best practices make them safer, such as:

- Not checking in Secret object definition files to source code repositories.
- [Enabling Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets so they are stored encrypted in etcd.

The way Kubernetes handles Secrets also helps:

- A Secret is only sent to a node if a pod on that node requires it.
- The kubelet stores the Secret in a tmpfs so that it is not written to disk storage.
- Once the pod that depends on the Secret is deleted, the kubelet deletes its local copy of the Secret data as well.

Read about the [protections](https://kubernetes.io/docs/concepts/configuration/secret/#protections) and [risks](https://kubernetes.io/docs/concepts/configuration/secret/#risks) of using Secrets.

Having said that, there are better ways of handling sensitive data like passwords in Kubernetes, such as tools like Helm Secrets and [HashiCorp Vault](https://www.vaultproject.io/). I hope to make a lecture on these in the future.
