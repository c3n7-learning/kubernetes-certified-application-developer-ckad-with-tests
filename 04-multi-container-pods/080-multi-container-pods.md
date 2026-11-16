# Multi-Container Pods

Sometimes, you may want to have 2 containers running side by side

- For example, a web-server and a main app.
- You may want a web-server (nginx) to run side-by-side against the main app
- You may also want the two to be started and stopped at the same time
- You may also want them to share the same lifecycle, network and storage

To create a multi-container pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080
    - name: main-app
      image: main-app
```
