# Service Account

Is linked to other security control in k8s

- For CKAD, you only need to know how to work with `ServiceAccounts`

There are two types of accounts in k8s

- User accounts (used by humans)
- Service Accounts (used by applications/bots/etc)

Service accounts are used by

- Prometheus pulls the k8s API for performance metrics
- Jenkins to deploy applications on a k8s cluster
- For an application to query the k8s api, it needs a srvice account

## Creating Service Accounts

Create a service account:

```shell
$ kubectl create serviceaccount dashboard-sa
```

Get the service accounts:

```shell
$ kubectl get serviceaccount
```

Describe the created service account:

- The application will need the Token to authenticate against the k8s api

```shell
$ kubectl describe serviceaccount dashboard-ca

Name:                dashboard-sa
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   dashboard-sa-token-kbbdm
Tokens:              dashboard-sa-token-kbbdm
Events:              <none>
```

The token though is stored as a secret object

- In this case, named `dashboard-sa-token-kbbdm`

To view the token:

```shell
$ kubectl describe secret dashboard-sa-token-kbbdm

Name:         dashboard-sa-token-kbbdm
Namespace:    default
Labels:       <none>

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  7 bytes
token:
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3
```

This token can then be used to authenticate against k8s api:

```shell
$ curl https://192.168.56.70:6443/api -insecure
     --header "Authorization: Bearer eyJhbGci..."
```

What if the application is stored on k8s itself?

- We then don't have to go through this process of bearer tokens
- We can then just mount the service account secret inside the application

## Default Service Account

Each namespace has its own `default` service account:

- Its token is mounted onto pods as a volume mount

```shell
$ kubectl get serviceaccount

NAME           AGE
dashboard-sa   4m9s
default        2d3h
```

How it is mounted once the pod is created:

```shell
$ kubectl describe pod my-kubernetes-dashboard

Name:         my-kubernetes-dashboard
Namespace:    default
Annotations:  <none>
Status:       Running
IP:           10.244.0.15
Containers:
  nginx:
    Image:        my-kubernetes-dashboard
Mounts:
    /var/run/secrets/kubernetes.io/serviceaccount from default-token-j4hkv (ro)
Conditions:
  Type          Status
Volumes:
  default-token-j4hkv:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-j4hkv
    Optional:    false
```

If you view the contents of that token, you'll see the values:

- It is heavily restricted:
- The token is only allowed to do basic k8s api queries.

```shell
$ kubectl exec -it my-kubernetes-dashboard -- ls /var/run/secrets/kubernetes.io/serviceaccount

ca.crt  namespace  token

$ kubectl exec -it my-kubernetes-dashboard cat /var/run/secrets/kubernetes.io/serviceaccount/token

eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tajRoa3YiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjcxZGM4YWExLTU2MGMtMTFlOC04YmI0LTA4MDAyNzkzMTA3MiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNj
```

## Editing the Service Account

To use a different service account in a pod rather than the default:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  serviceAccountName: dashboard-sa
```

If we don't want the service account to be mounted by default:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  automountServiceAccountToken: false
```

## Service Accounts (k8s `1.22`, `1.44` Update)

If you decode the token on `jwt.io`, you should see something like:

```json
{
  "aud": ["https://kubernetes.default.svc.cluster.local", "k3s"],
  "exp": 1826299841,
  "iat": 1794763841,
  "iss": "https://kubernetes.default.svc.cluster.local",
  "jti": "171c9b68-12e9-4fbe-bc8d-f54ac3043737",
  "kubernetes.io": {
    "namespace": "default",
    "node": {
      "name": "k3d-mycluster-agent-0",
      "uid": "fc680a49-4998-415b-aca3-73da031b062b"
    },
    "pod": {
      "name": "ubuntu-sleeper",
      "uid": "d560eeeb-8f6b-465b-a4b8-c31d9551ad50"
    },
    "serviceaccount": {
      "name": "default",
      "uid": "decc3c1c-a967-4ff1-86fb-c12c3bed85df"
    },
    "warnafter": 1794767448
  },
  "nbf": 1794763841,
  "sub": "system:serviceaccount:default:default"
}
```

### (pre v1.22) KEP 1205 - Bound Service Account Tokens

Kubernetes already provisions JWTs to workloads. This functionality is on by default and thus widely deployed. The current workload JWT system has serious issues:

1. **Security:** JWTs are not audience bound. Any recipient of a JWT can masquerade as the presenter to anyone else.
2. **Security:** The current model of storing the service account token in a Secret and delivering it to nodes results in a broad attack surface for the Kubernetes control plane when powerful components are run - giving a service account a permission means that any component that can see that service account's secrets is at least as powerful as the component.
3. **Security:** JWTs are not time bound. A JWT compromised via 1 or 2, is valid for as long as the service account exists. This may be mitigated with service account signing key rotation but is not supported by client-go and not automated by the control plane and thus is not widely deployed.
4. **Scalability:** JWTs require a Kubernetes secret per service account.

### (post v1.22) `TokenRequestAPI` tokens

They are:

- Audience bound
- Time bound
- Object bound

```shell
$ kubectl get pod ubuntu-sleeper -o yaml
```

You'll see

- `spec.containers.volumeMounts.mountPath`, and
- `spec.volumes.projected.sources.serviceAccountToken.expirationSeconds`
- It communicates with the token request api, and gets a token for the pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-11-15T17:30:40Z"
  generation: 1
  labels:
    run: ubuntu-sleeper
  name: ubuntu-sleeper
spec:
  containers:
    - command:
        - sleep
        - "10000"
      image: ubuntu
      name: ubuntu-sleeper
      volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-cfm2g
          readOnly: true
  serviceAccount: default
  serviceAccountName: default
  volumes:
    - name: kube-api-access-cfm2g
      projected:
        defaultMode: 420
        sources:
          - serviceAccountToken:
              expirationSeconds: 3607
              path: token
          - configMap:
              items:
                - key: ca.crt
                  path: ca.crt
              name: kube-root-ca.crt
          - downwardAPI:
              items:
                - fieldRef:
                    apiVersion: v1
                    fieldPath: metadata.namespace
                  path: namespace
```

### (v1.24) KEP-2799: Reduction of Secret-based Service Account Tokens

Previously,

- When you create a service account, a secret was created and mounted to any pod that uses it
- This secret had no expiry (changed in v1.22)

With v1.24,

- Creating a service account does not create a service account automatically:

You first create the service account:

```shell
$ kubectl create serviceaccount dashboard-sa
```

Then create a token for the service account:

- The token will be printed on to the screen

```shell
$ kubectl create token dashboard-sa

eyJhbGciOiJSUzI1NiI...
```

If you decode it on jwt.io

- You'll see it has an expiry date

```json
{
  "aud": ["https://kubernetes.default.svc.cluster.local", "k3s"],
  "exp": 1794768296,
  "iat": 1794764696,
  "iss": "https://kubernetes.default.svc.cluster.local",
  "jti": "2f9166b8-e90a-42a8-9108-7fc36ec84cb8",
  "kubernetes.io": {
    "namespace": "default",
    "serviceaccount": {
      "name": "dashboard-sa",
      "uid": "5c13f5ef-028f-4fb1-a8ab-95c55518d445"
    }
  },
  "nbf": 1794764696,
  "sub": "system:serviceaccount:default:dashboard-sa"
}
```

## Creating non-expiring tokens, the old way

You create a spec file with the type `kubernetes.io/service-account-token`:

- The name of the service account in `kubernetes.io/service-account.name`

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: dashboard-sa
```

The created token will never expire. It is a frowned-upon practice though:

> You should only create a service account token Secret object if you can't use the `TokenRequest` API to obtain a token, and the security exposure of persisting a non-expiring token credential in a readable API object is acceptable to you.

## References

**KEP 1205 - Bound Service Account Tokens**

- https://github.com/kubernetes/enhancements/tree/master/keps/sig-auth/1205-bound-service-account-tokens

**KEP-2799: Reduction of Secret-based Service Account Tokens**

- https://github.com/kubernetes/enhancements/tree/master/keps/sig-auth/2799-reduction-of-secret-based-service-account-token
- https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets
- https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/
