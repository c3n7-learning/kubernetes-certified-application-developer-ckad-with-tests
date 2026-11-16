# Define, build and modify container images

## Introduction

What am I containerizing?

- An application built with Flask

Steps if we were to deploy it manually:

1. OS - Ubuntu
2. Update apt repo
3. Install dependencies using apt
4. Install Python dependencies using pip
5. Copy source code to /opt folder
6. Run the web server using "flask" command

How to create my own image?

```Dockerfile
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

Build it:

```shell
$ docker build Dockerfile -t karanimwenda/my-custom-app
```

Push it to docker registry

```shell
$ docker push karanimwenda/my-custom-app
```

## Dockerfile

It follows an `<INSTRUCTION> <argument>` format

```Dockerfile
FROM Ubuntu                   # start from a base OS or another image

RUN apt-get update            # install all dependencies
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code       # Copy source code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run    # Specify entry point that will be run
```

## Layered Architecture:

Each line in the Dockerfile creates a layer that builds on top of the previous layer

```Dockerfile
FROM Ubuntu                                               # Layer 1: Base Ubuntu layer (120MB)

RUN apt-get update && apt-get install python              # Layer 2: Changes in apt packages (306MB)

RUN pip install flask flask-mysql                         # Layer 3: Changes in pip packages (6.3MB)

COPY . /opt/source-code                                   # Layer 4: Source code (229 B)

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run    # Layer 5: Update Entrypoint with "flask" command (0 B)
```

You can see sizes after running a build via:

```shell
docker history karanimwenda/simple-webapp
```

The benefit is if a build fails at layer 3, when you resume/retry, the build starts at layer 3

- This helps us have faster builds
- We don't have to wait for builds to start from scratch everytime

## What can you containerize?

You can containerize almost all applications

- Browsers
- Utilities like curl
- Applications like Spotify
- Anything!!
