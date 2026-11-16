# Commands and Arguments in Docker

Say you run an ubuntu image

```bash
$ docker run ubuntu
```

It would run and exit immediately

```bash
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

```bash
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

```bash
$ docker run ubuntu-sleeper sleep 10
```

But we want to run it like:

```bash
$ docker run ubuntu-sleeper 10
```

## `ENTRYPOINT`

Here comes: `ENTRYPOINT`

```Dockerfile
FROM Ubuntu
ENTRYPOINT ["sleep"]
```

Now the parameters can be appended:

```bash
$ docker run ubuntu-sleeper 10
```

What if we want to specify a default value?

```Dockerfile
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Now we can run:

```bash
$ docker run ubuntu-sleeper 10
```

Or:

- Will use the 5 seconds

```bash
$ docker run ubuntu-sleeper
```

How to modify the entrypoint at runtime?

```bash
$ docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```
