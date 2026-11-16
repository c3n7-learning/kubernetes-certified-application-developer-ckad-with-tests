# Environment Variables

Environment variables let you configure a container at startup. In Docker you pass them with the `-e` flag; in Kubernetes you declare them under the `env` property of the container in the Pod definition.

## Docker

```bash
docker run -e APP_COLOR=pink simple-webapp-color
```

## Kubernetes

`pod-definition.yaml`

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
```

## How They Map

| Docker                      | Kubernetes                      |
| --------------------------- | ------------------------------- |
| `-e APP_COLOR=pink`         | `env:` list under the container |
| `APP_COLOR` (variable name) | `name: APP_COLOR`               |
| `pink` (variable value)     | `value: pink`                   |

## Notes

- `env` is an **array**, so each variable starts with a `-`.
- Each item has a `name` and a `value`.
- Add more variables by adding more `- name` / `value` entries.

## Kubernetes: Ways to Set Environment Variables

### 01 — Plain Key Value

```yaml
env:
  - name: APP_COLOR
    value: pink
```

### 02 — ConfigMap

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

### 03 — Secrets

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: APP_COLOR
```
