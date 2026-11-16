# Docker vs ContainerD

Older k8s docs reference docker, new docs reference **containerd**

## A bit of History

Introduction

- Containerization was made popular with docker
- There were other solutions like `rkt` (rocket)
- Then k8s came and primarily worked with docker

Cross-compatibility:

- Others wanted to run on k8s as well (not just docker)
- k8s introduced the **Container Runtime Interface (CRI)** that allowed any vendor to work as a container runtime for k8s, as long as they adhere to the **Open Container Initiative (OCI)**
- The OCI defines specifications on an `imagespec` and `runtimespec`
- `imagespec` defines how an image should be built
- `runtimespec` defines how any container runtime should be developed

But

- Docker wasn't built to support the CRI standard
- k8s developed a `dockershim` to help bridge the gap between docker and the CRI

Docker is not just a container runtime. It contains:

- CLI
- API
- BUILD
- VOLUMES
- AUTH
- SECURITY runtime ()
- Container Runtime (runc)
- ContainerD: The deamon that managed `runc` (which is CRI compatible)

Thus, ContainerD can be used a separate runtime, separate from docker

- Maintaining `dockershim` was too much work, so it was discotinued in k8s v1.24
- Docker containers continue to run on k8s due to ContainerD, so no breaking change

## ContainerD

### Introduction

- Is now maintained by its own, by the CNCF
- Can be installed on its own
- https://github.com/containerd/containerd

### CLI (`ctr`)

Is not the most user friendly CLI, but is used to debug ContainerD

Pulling Images:

```bash
ctr images pull docker.io/library/redis:alpine
```

Running the image

```bash
ctr run docker.io/library/redis:alpine redis
```

### Better alternative (`nerdctl`)

Is the recommended alternative

- Is a docker-like CLI for containerd
- Supports features like docker, e.g. Docker compose
- Supports newest features in containerd, for example:
  - encrypted container images
  - lazy pulling
  - p2p image distrubution
  - image signing and verifying
  - namespaces in k8s

Run a container:

```bash
# Docker
$ docker run --name redis redis:alpine

# nerdctl
$ nerdctl run --name redis redis:alpine
```

Or:

```bash
# Docker
$ docker run --name webserver -p 80:80 -d nginx

# nerdctl
$ nerdctl run --name webserver -p 80:80 -d nginx
```

## CLI - `crictl`

`crictl` provides a CLI for CRI-compatible container runtimes

- Installed superately
- Used to inspect and debug container runtimes
  - Not to create containers ideally
- Works across different runtimes (rkt, containerd, etc)

Examples

```bash
$ crictl pull busybox

$ crictl images

$ crictl ps -a

$ crictl exec -i -t thecontainerid ls

$ crictl logs thecontainerid ls

# List k8s pods
$ crictl pods
```

Retrieve debugging information

| docker cli | crictl                | Description                                                          | Unsupported Features                      |
| ---------- | --------------------- | -------------------------------------------------------------------- | ----------------------------------------- |
| `attach`   | `attach`              | Attach to a running container                                        | `--detach-keys`, `--sig-proxy`            |
| `exec`     | `exec`                | Run a command in a running container                                 | `--privileged`, `--user`, `--detach-keys` |
| `images`   | `images`              | List images                                                          |                                           |
| `info`     | `info`                | Display system-wide information                                      |                                           |
| `inspect`  | `inspect`, `inspecti` | Return low-level information on a container, image or task           |                                           |
| `logs`     | `logs`                | Fetch the logs of a container                                        | `--details`                               |
| `ps`       | `ps`                  | List containers                                                      |                                           |
| `stats`    | `stats`               | Display a live stream of container(s) resource usage statistics      | Column: NET/BLOCK I/O, PIDs               |
| `version`  | `version`             | Show the runtime (Docker, ContainerD, or others) version information |                                           |

Perform Changes

| docker cli | crictl               | Description                                    | Unsupported Features                                                              |
| ---------- | -------------------- | ---------------------------------------------- | --------------------------------------------------------------------------------- |
| `create`   | `create`             | Create a new container                         |                                                                                   |
| `kill`     | `stop` (timeout = 0) | Kill one or more running container             | `--signal`                                                                        |
| `pull`     | `pull`               | Pull an image or a repository from a registry  | `--all-tags`, `--disable-content-trust`                                           |
| `rm`       | `rm`                 | Remove one or more containers                  |                                                                                   |
| `rmi`      | `rmi`                | Remove one or more images                      |                                                                                   |
| `run`      | `run`                | Run a command in a new container               |                                                                                   |
| `start`    | `start`              | Start one or more stopped containers           | `--detach-keys`                                                                   |
| `stop`     | `stop`               | Stop one or more running containers            |                                                                                   |
| `update`   | `update`             | Update configuration of one or more containers | `--restart`, `--blkio-weight` and some other resource limit not supported by CRI. |

You need to configure which runtime it needs to interface with

- By default, its checks the following in descending order
  - `unix:///var/run/dockershim.sock` or
  - `unix:///run/containerd/containerd.sock` or
  - `unix:///run/crio/crio.sock` or
  - `unix:///var/run/cri-dockerd.sock`
- To override
  - Use `crictl --runtime-endpoint`
  - Or `export CONTAINER_RUNTIME_ENDPOINT`
