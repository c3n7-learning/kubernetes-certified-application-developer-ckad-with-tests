# Liveness Probes

What happens when a container stays alive, but the application is not working?

- A liveness probes helps with this
- Is used to test if the application in the container is actually healthy
- If it is not, the container is destroyed and re-created

It is defined in the pod definition file

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      livenessProbe:
        httpGet:
          path: /api/healthy
          port: 8080
```

You have similar options as the `readinessProbe`:

- If your app exposes an HTTP health endpoint, with timing controls:

```yaml
livenessProbe:
  httpGet:
    path: /api/healthy
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 8
```

- If you're checking a database or other TCP service (e.g., MySQL):

```yaml
livenessProbe:
  tcpSocket:
    port: 3306
```

- If you want to run a command inside the container to check health:

```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /app/is_healthy
```
