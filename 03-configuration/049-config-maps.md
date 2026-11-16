# ConfigMaps

The values are specified in key->value pairs

We could specify env values directly in a pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink
        - name: APP_MODE
          value: prod
```

We could do this in a config map though. Then inject the config map into the pod:

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```

## Creating Config Maps

Assume you have the configs:

```
APP_COLOR: blue
APP_MODE: prod
```

### Imperatively

Structure:

```shell
$ kubectl create configmap <config-name> \
    --from-literal=<key>=<value>
```

Example:

```shell
$ kubectl create configmap app-config \
    --from-literal=APP_COLOR=blue \
    --from-literal=APP_MODE=prod
```

With configs from file

```shell
$ kubectl create configmap app-config \
    --from-file=app_config.properties
```

### Declaratively

Create a definition file

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

Then create it

```shell
$ kubectl create -f config-values.yaml
```

## Viewing ConfigMaps

Run

```shell
$ kubectl get configmaps
```

View the data in the configmap

```shell
$ kubectl describe configmaps
```

## ConfigMap in Pods

To inject the config map into a pod:

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```

You can inject the config data as files in a volume as well:

```yaml
volumes:
  - name: app-config-volume
    configMap:
      name: app-config
```
