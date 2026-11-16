# Commands and Arguments in Docker

Say you run an ubuntu image

```shell
$ docker run ubuntu
```

It would run and exit immediately

```shell
$ docker ps
# nothing
$ docker ps -a
# the ubuntu image
```

Why?

- Containers are not meant to run operating systems, in ideal situations
- They are mean to run one thing only
- Is defined via the `CMD` operative

How to override the command for an image?

```shell
$ docker run ubuntu [COMMAND]
$ docker run ubuntu sleep 5
```

How to make this permanent?

- Create a custom image

```Dockerfile
FROM Ubuntu
CMD sleep 5

# or CMD ["sleep", "5]
# or CMD ["sleep 5" ]
```

How to change the number fo seconds it sleeps?

```shell
$ docker run ubuntu-sleeper sleep 10
```

But we want to run it like:

```shell
$ docker run ubuntu-sleeper 10
```

## `ENTRYPOINT`

Here comes: `ENTRYPOINT`

```Dockerfile
FROM Ubuntu
ENTRYPOINT ["sleep"]
```

Now the parameters can be appended:

```shell
$ docker run ubuntu-sleeper 10
```

What if we want to specify a default value?

```Dockerfile
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Now we can run:

```shell
$ docker run ubuntu-sleeper 10
```

Or:

- Will use the 5 seconds

```shell
$ docker run ubuntu-sleeper
```

How to modify the entrypoint at runtime?

```shell
$ docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```
