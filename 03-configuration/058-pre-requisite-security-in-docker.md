# Pre-Requisite - Security in Docker

## Process Isolation

Assume we run:

```shell
docker run ubuntu sleep 3600
```

Containers are not completely isolated from the host

- Containers share the same kernel
- They are isolated by namespaces
- All processes run in docker containers can be seen in the host, but the docker container can only see its own

## Users

Docker has a set of users, root and non-root

- By default, docker runs processes in contrainers as the root user

If you want to run a processes in a container as root:

- Imperatively:

```shell
$ docker run --user=1001 ubuntu sleep 3600
```

- Declaratively:

```Dockerfile
FROM ubuntu

USER 1000
```

### The Docker Root User

Is the user in docker that is `root` have the same rights as the docker-host machine's root user?

- Nope. The root user in docker is not the same as the host's root user
- Docker uses linux capabilities to do this

A root user's capabilities in a linux host can be seen in `/usr/include/linux/capability.h`

- https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h
- Docker runs with a limited set of capabilities.

Linux root user's capabilities

- CHOWN
- DAC
- KILL
- SETFCAP
- SETPCAP
- SETGID
- SETUID
- NET_BIND
- NET_RAW
- MAC_ADMIN
- BROADCAST
- NET_ADMIN
- SYS_ADMIN
- SYS_CHROOT
- AUDIT_WRITE
- MANY MORE...

Docker root user's capabilities

- MAC_ADMIN
- BROADCAST
- NET_ADMIN
- SYS_ADMIN
- MANY MORE...

To remove/add a capability to a docker container:

```shell
$ docker run --cap-add MAC_ADMIN ubuntu
$ docker run --cap-drop KILL ubuntu
```

To run with all capabilities

```shell
$ docker run --privileged ubuntu
```
