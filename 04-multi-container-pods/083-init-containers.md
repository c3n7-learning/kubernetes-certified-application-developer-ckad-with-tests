# Init Containers

In a multi-container pod, each container is expected to run a process that stays alive as long as the Pod's lifecycle. For example, in the multi-container pod we discussed earlier, which has a web application and a logging agent, both containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If either of them fails, the Pod restarts.

At times, though, you may want to run a process that runs to completion in a container. For example:

- A process that pulls code or a binary from a repository that the main web application will use. This task runs only once, when the Pod is first created.
- A process that waits for an external service or database to be up before the actual application starts.

That's where `initContainers` come in.

An initContainer is configured in a Pod like all other containers, except that it is specified inside an `initContainers` section, like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
  initContainers:
    - name: init-myservice
      image: busybox
      command:
        [
          "sh",
          "-c",
          "git clone <some-repository-that-will-be-used-by-application> ;",
        ]
```

When a Pod is first created, the initContainer runs, and its process must run to completion before the real container hosting the application starts.

You can configure multiple initContainers as well, just as with multi-container Pods. In that case, each init container runs one at a time, in sequential order.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the init container succeeds.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup myservice; do echo waiting for myservice; sleep 2; done;",
        ]
    - name: init-mydb
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup mydb; do echo waiting for mydb; sleep 2; done;",
        ]
```

Read more about initContainers [here](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/), and try out the upcoming practice test.
