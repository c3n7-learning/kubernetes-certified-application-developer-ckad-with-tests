# Jobs

Not every workload runs forever. Web servers, applications, and databases run until someone takes them down, but **batch jobs** perform a specific task and then exit.

## A single operation in Docker

```bash
$ docker run ubuntu expr 3 + 2
5
$ docker ps -a
CONTAINER ID   IMAGE    COMMAND        CREATED          STATUS                     NAMES
7bf64d3cc13a   ubuntu   "expr 3 + 2"   14 seconds ago   Exited (0) 13 seconds ago  objective_chebyshev
```

The container exits with code `0`, meaning the command succeeded.

## A single operation in Kubernetes

A naive approach is a plain Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: math-pod
spec:
  containers:
    - name: math-add
      image: ubuntu
      command: ["expr", "3", "+", "2"]
```

```bash
$ kubectl create -f pod-definition.yaml
$ kubectl get pods
NAME       READY   STATUS      RESTARTS   AGE
math-pod   0/1     Completed   3          1d
```

Note the restarts. Kubernetes assumes a container should live forever, so it keeps restarting it, even after it completes successfully. This behavior is controlled by `spec.restartPolicy`:

- `Always` (default): restart whenever the container exits
- `OnFailure`: restart only on a non-zero exit code
- `Never`: don't restart

For run-to-completion tasks, set `restartPolicy` to `Never` or `OnFailure`, or better, use a Kubernetes **Job**.

### Kubernetes Jobs

- Replica sets ensure a number of pods keep running all the times
- Jobs ensure a task is performed successfully to completion

To create a job:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ["expr", "3", "+", "2"]
    restartPolicy: Never
```

Now run it:

```bash
$ kubectl create -f job-definition.yaml
$ kubectl get jobs
NAME           DESIRED   SUCCESSFUL   AGE
math-add-job   1         1            38s

$ kubectl get pods
NAME                 READY   STATUS      RESTARTS   AGE
math-add-job-l87pn   0/1     Completed   0          2m
```

- Now the job runs successfully, and the pod is not restarted.
- What about the output of the pod?

```bash
$ kubectl logs math-add-job-ld87pn
5
```

To delete a job

```bash
$ kubectl delete job math-add-job
job.batch "math-add-job" deleted
```

## Multiple Pods

To run multiple pods, update `spec.completions`:

- By default, the pods are created one after another
- The second pod is created only after the first on finishes

```yaml
# job-definition.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ["expr", "3", "+", "2"]
      restartPolicy: Never
```

```bash
$ kubectl create -f job-definition.yaml

$ kubectl get jobs
NAME           DESIRED   SUCCESSFUL   AGE
math-add-job   3         3            38s

$ kubectl get pods
NAME                 READY   STATUS      RESTARTS   AGE
math-add-job-25j9p   0/1     Completed   0          2m
math-add-job-87g4m   0/1     Completed   0          2m
math-add-job-d5z95   0/1     Completed   0          2m
```

- `completions: 3` tells the Job to run until 3 pods have finished successfully
- By default, pods are created one after the other, not in parallel

## Pod Failures

If a pod fails, the Job creates a new one, and keeps going until the desired number of completions is reached.

```yaml
# job-definition.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: random-error-job
spec:
  completions: 3
  template:
    spec:
      containers:
        - name: random-error
          image: kodekloud/random-error
      restartPolicy: Never
```

```bash
$ kubectl create -f job-definition.yaml

$ kubectl get jobs
NAME               DESIRED   SUCCESSFUL   AGE
random-error-job   3         3            38s

$ kubectl get pods
NAME                     READY   STATUS      RESTARTS   AGE
random-exit-job-ktmtt    0/1     Completed   0
random-exit-job-sdsrf    0/1     Error       0
random-exit-job-wwqbn    0/1     Completed   0
random-exit-job-fkhfn    0/1     Error       0
random-exit-job-fvf5t    0/1     Error       0
random-exit-job-nmghp    0/1     Completed   0
```

- The image exits with a random success or failure
- With `restartPolicy: Never`, failed pods stay as `Error` and the Job spins up replacements
- Here, 6 pods were needed to get 3 successful completions

## Parallelism

## Parallelism

By default, a Job creates pods one at a time. Set `parallelism` to run several pods at once.

```yaml
# job-definition.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: random-error-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: random-error
          image: kodekloud/random-error
      restartPolicy: Never
```

```bash
$ kubectl create -f job-definition.yaml

$ kubectl get jobs
NAME               DESIRED   SUCCESSFUL   AGE
random-error-job   3         2            38s

$ kubectl get pods
NAME                    READY   STATUS      RESTARTS
random-exit-job-ktmtt   0/1     Completed   0
random-exit-job-sdsrf   0/1     Error       0
random-exit-job-wwqbn   0/1     Completed   0
random-exit-job-fkhfn   0/1     Error       0
random-exit-job-fvf5t   0/1     Error       0
random-exit-job-nmghp   0/1     Completed   0
```

- `parallelism: 3` starts up to 3 pods at the same time
- If a pod fails, the Job creates a replacement
- The Job never runs more pods at once than are still needed to reach `completions`
