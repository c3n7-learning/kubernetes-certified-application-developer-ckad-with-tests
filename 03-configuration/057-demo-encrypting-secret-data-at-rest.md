# Demo: Encrypting Secret Data at Rest

We will follow this guide:

- https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

Let's create a generic secret:

```shell
$ kubectl create secret generic my-secret --from-literal=key1=supersecret
```

Let's see if it was created:

```
$ kubectl get secrets

NAME        TYPE     DATA   AGE
my-secret   Opaque   1      4s
```

Let's see it's yaml:

```shell
$ kubectl get secret my-secret -o yaml

apiVersion: v1
data:
  key1: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  creationTimestamp: "2026-11-15T07:14:37Z"
  name: my-secret
  namespace: default
  resourceVersion: "31069"
  uid: 46b89dfb-975b-4d88-a529-d6a8cb68f05b
type: Opaque
```

We could base64 decode that value:

```shell
$ echo 'c3VwZXJzZWNyZXQ=' | base64 --decode

supersecret
```

So you shouldn't commit this, as it can be decoded by anyone.

- We won't fix this. We'll only fix how it's stored in the `etcd` server

## How it is stored in `etcd`

Reference

- https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted

Install it:

```shell
# Debian/Ubuntu
$ apt-get install etcd-client

# MacOS
$ brew install etcd
```

Let's see how it's been stored:

```shell
ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/my-secret | hexdump -C
```

You'll see:

```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6d 79 2d 73 65 63  |s/default/my-sec|
00000020  72 65 74 0a 6b 38 73 00  0a 0c 0a 02 76 31 12 06  |ret.k8s.....v1..|
00000030  53 65 63 72 65 74 12 d0  01 0a b0 01 0a 09 6d 79  |Secret........my|
00000040  2d 73 65 63 72 65 74 12  00 1a 07 64 65 66 61 75  |-secret....defau|
00000050  6c 74 22 00 2a 24 63 66  34 65 35 34 63 65 2d 34  |lt".*$cf4e54ce-4|
00000060  30 37 65 2d 34 37 39 35  2d 61 61 33 62 2d 61 33  |07e-4795-aa3b-a3|
00000070  39 35 31 31 37 35 62 65  62 61 32 00 38 00 42 08  |951175beba2.8.B.|
00000080  08 8e be e5 d7 06 10 00  8a 01 61 0a 0e 6b 75 62  |..........a..kub|
00000090  65 63 74 6c 2d 63 72 65  61 74 65 12 06 55 70 64  |ectl-create..Upd|
000000a0  61 74 65 1a 02 76 31 22  08 08 8e be e5 d7 06 10  |ate..v1"........|
000000b0  00 32 08 46 69 65 6c 64  73 56 31 3a 2d 0a 2b 7b  |.2.FieldsV1:-.+{|
000000c0  22 66 3a 64 61 74 61 22  3a 7b 22 2e 22 3a 7b 7d  |"f:data":{".":{}|
000000d0  2c 22 66 3a 6b 65 79 31  22 3a 7b 7d 7d 2c 22 66  |,"f:key1":{}},"f|
000000e0  3a 74 79 70 65 22 3a 7b  7d 7d 42 00 12 13 0a 04  |:type":{}}B.....|
000000f0  6b 65 79 31 12 0b 73 75  70 65 72 73 65 63 72 65  |key1..supersecre|
00000100  74 1a 06 4f 70 61 71 75  65 1a 00 22 00 0a        |t..Opaque.."..|
0000010e
```

- You can see `supersecret` somewhere there.
- That's what we want to resolve.

## Encryption at Rest

Let's determine whether encryption at rest is already enabled

- https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#determining-whether-encryption-at-rest-is-already-enabled

```shell
$ ps -aux | grep kube-api | grep "encryption-provider-config"

# Nothing
```

In a kube-admin setup:

- You won't see the `--encryption-provider-config` option

```shell
$ cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

## Enabling encryption at rest

We'll create a configuration and apply it

- Available encryption providers: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#providers
- The `resources.resources.providers` is a list. The first provider found is the one used for encryption.
- If we have `identity` at the top, then it'll be stored in plain text

```yaml
---
#
# CAUTION: this is an example configuration.
#          Do not use this for your own cluster!
#
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources: # INFO: resources that you want to encrypt
      - secrets
      - configmaps
      - pandas.awesome.bears.example # a custom resource API
    providers: # INFO: the provider that we'll use to encrypt stuff
      # This configuration does not provide data confidentiality. The first
      # configured provider is specifying the "identity" mechanism, which
      # stores resources as plain text.
      #
      - identity: {} # plain text, in other words NO encryption
      - aesgcm:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
            - name: key2
              secret: dGhpcyBpcyBwYXNzd29yZA==
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
            - name: key2
              secret: dGhpcyBpcyBwYXNzd29yZA==
      - secretbox:
          keys:
            - name: key1
              secret: YWJjZGVmZ2hpamtsbW5vcHFyc3R1dnd4eXoxMjM0NTY=
  - resources:
      - events
    providers:
      - identity: {} # do not encrypt Events even though *.* is specified below
  - resources:
      - "*.apps" # wildcard match requires Kubernetes 1.27 or later
    providers:
      - aescbc:
          keys:
            - name: key2
              secret: c2VjcmV0IGlzIHNlY3VyZSwgb3IgaXMgaXQ/Cg==
  - resources:
      - "*.*" # wildcard match requires Kubernetes 1.27 or later
    providers:
      - aescbc:
          keys:
            - name: key3
```

### Encrypt your data

Generate the encryption key;

```shell
$ head -c 32 /dev/urandom | base64

4vrcam35HOqY0M7xKyAutg6LMF9KPYG6QR87DZcFzV0=
```

Let's create a custom encryption config:

- Save it at `/etc/kubernetes/enc/enc.yaml` on the control plane node

```yaml
---
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: 4vrcam35HOqY0M7xKyAutg6LMF9KPYG6QR87DZcFzV0=
      - identity: {} # this fallback allows reading unencrypted secrets;
          # for example, during initial migration
```

Use the new encryption configuration file:

- Edit the manifest for the `kube-apiserver` static pod: `/etc/kubernetes/manifests/kube-apiserver.yaml` so that it is similar to

```yaml
---
#
# This is a fragment of a manifest for a static Pod.
# Check whether this is correct for your cluster and for your API server.
#
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.20.30.40:443
  creationTimestamp: null
  labels:
    app.kubernetes.io/component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # add this line
    volumeMounts:
    ...
    - name: enc                           # add this line
      mountPath: /etc/kubernetes/enc      # add this line
      readOnly: true                      # add this line
    ...
  volumes:
  ...
  - name: enc                             # add this line
    hostPath:                             # add this line
      path: /etc/kubernetes/enc           # add this line
      type: DirectoryOrCreate             # add this line
  ...
```

On save, the api server will restart:

```shell
$ kubectl get pods
The connection to the server controlplane:6443 was refused - did you specify the right host or port?

$ crictl pods
```

Once the api-server is back up, check if encryption has been configured successfully:

```shell
$ ps -aux | grep kube-api | grep encryption
```

## Let's test out the encryption

Create another secret

```shell
$ kubectl create secret generic my-secret-2 --from-literal=key2=topsecret
secret/my-secret-2 created
```

```shell
ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/my-secret-2 | hexdump -C
```

You'll see:

```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6d 79 2d 73 65 63  |s/default/my-sec|
00000020  72 65 74 2d 32 0a 6b 38  73 3a 65 6e 63 3a 61 65  |ret-2.k8s:enc:ae|
00000030  73 63 62 63 3a 76 31 3a  6b 65 79 31 3a 3f 29 d2  |scbc:v1:key1:?).|
00000040  b4 19 ba 58 ac da 1f ae  21 d2 50 c4 f2 bd ce c5  |...X....!.P.....|
00000050  ef af ed fa 15 50 4d 22  59 1a 80 e4 73 d0 3b 56  |.....PM"Y...s.;V|
00000060  61 66 8c 02 09 ef 15 47  77 49 d0 74 73 98 65 29  |af.....GwI.ts.e)|
00000070  b3 d8 07 f0 8a e9 dd ca  47 68 65 1f f6 06 d5 2e  |........Ghe.....|
00000080  0b e8 a2 3a d3 fc c2 55  9b b3 67 c8 e2 3d 8e 15  |...:...U..g..=..|
00000090  94 da 6e aa ee 11 bc 26  38 e2 80 91 d2 7f 05 2f  |..n....&8....../|
000000a0  91 ab ad c7 79 dd 63 68  b0 29 7c 49 57 ba 19 5d  |....y.ch.)|IW..]|
000000b0  a8 b7 8c 67 a6 8f 2b dd  3c 48 7d cc c2 ac 85 24  |...g..+.<H}....$|
000000c0  e6 18 17 d2 ef 5e 0c dd  ee ac 50 53 cc 86 8e 0b  |.....^....PS....|
000000d0  80 14 86 80 0c b6 ef 73  db b6 f4 16 79 ac 6d 0e  |.......s....y.m.|
000000e0  80 d1 10 bf b0 4c 74 54  be af f4 54 6a d2 51 ac  |.....LtT...Tj.Q.|
000000f0  7c 0e f0 7b 6f fb 27 38  14 70 d5 b9 1f fc 7c ce  ||..{o.'8.p....|.|
00000100  10 43 6d fb 50 08 24 37  9d 11 df 80 c5 1a fe ff  |.Cm.P.$7........|
00000110  6d b7 bd c5 67 32 5e 77  b4 b6 1c 3f c9 4f 62 37  |m...g2^w...?.Ob7|
00000120  2c 8f e7 94 10 a0 68 82  fd ca 1b 6a b1 3e f6 13  |,.....h....j.>..|
00000130  b0 90 a4 87 06 c9 8c a1  eb d3 58 49 f1 0a        |..........XI..|
0000013e
```

All secrets (created after this change), will be encrypted.

- To encrypt historical secrets

```shell
# Run this as an administrator that can read and write all Secrets
$ kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```
