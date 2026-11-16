# Secrets

Are used to store sensitive information, like passwords, or keys

- They are like configsmaps, but they are stored in a hashed format

## Creating secrets

### Imperatively:

```shell
$ kubectl create secret generic <secret-name> \
    --from-literal=<key>=<value>
```

Example:

```shell
$ kubectl create secret generic app-secret \
    --from-literal=DB_Host=mysql \
    --from-literal=DB_User=root \
    --from-literal=DB_Password=password
```

From file:

```shell
$ kubectl create secret generic <secret-name> \
    --from-file=<path-to-file>
```

Example:

```shell
$ kubectl create secret generic <secret-name> \
    --from-file=app_secret.properties
```

### Declaratively

Encode the values via base64, for example like:

```shell
$ echo -n 'mysql' | base64
$ echo -n 'root' | base64
$ echo -n 'passwrd' | base64
```

Which you can then use like

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
```

Then apply it

```shell
$ kubectl create -f secret-definition.yaml
```

## Viewing Secrets

To get all secrets in the namespace:

```shell
$ kubectl get secrets
```

To get the details of a secret (with the values hidden)

```shell
$ kubectl describe secrets
```

To view the hashed values as well

```shell
$ kubectl get secret app-secret -o yaml
```

To decode the values:

```shell
echo -n "bXlzcWw=" | base64 --decode
```

## Configuring it with a pod

To inject all the values of the secret as env values:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - secretRef:
            name: app-secret
```

To inject a single env value:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: DB_Password
          valueFrom:
            - secretKeyRef:
                name: app-secret
                key: DB_Password
```

To inject them as files in a volume

```yaml
volumes:
  - name: app-secret-volume
  secret:
    secretName: app-secret
```

- Each env value will be created as a file

```shell
$ ls /opt/app-secret-volumes
DB_host DB_Password DB_User

$ cat /opt/app-secret-volums/DB_Password
passwrd
```
