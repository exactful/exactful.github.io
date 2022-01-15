---
layout: post
title: Getting started with Docker in 6 minutes
category: docker
comments: true
---

This post covers the commands that I found most useful when starting to learn [Docker](https://www.docker.com/get-started) from installation to managing images and containers.

The first thing to do is install the Docker engine on your host platform.  In my case, that's [Ubuntu Mate 21.10](https://ubuntu-mate.org/download/) running on a [Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/) in the corner of my office. Docker provides [detailed instructions](https://docs.docker.com/engine/install/) on how to do this by platform and Linux distribution.

Once Docker is installed, we can start a Docker container from a Docker image.  This can be an image you've built yourself or one of the 8.4 million images available on [Docker Hub](https://hub.docker.com/search?q=&type=image).  From [Redis](https://hub.docker.com/_/redis) to [Jenkins](https://hub.docker.com/r/jenkins/jenkins) to [WordPress](https://hub.docker.com/_/wordpress), there's an image for it on the hub.

In the following example, the `docker run` command is used to create and start a container from an image of Debian, the Linux distribution.  Since the image doesn't already exist locally, Docker automatically checks the hub and tries to download it from there:

```python
$ docker run debian

Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
1c47a4233665: Pull complete
Digest: sha256:4d6ab716de467aad58e91b1b720f0badd7478847ec7a18f66027d0f8a329a43c
Status: Downloaded newer image for debian:latest
```

The image is downloaded and a container starts but immediately exits as it has nothing to do. You can verify it started and then stopped using the `docker ps` command with the `-a` flag to return all containers, not just the running ones:

```python
$ docker ps -a

CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS                      PORTS     NAMES
16223809b3db   debian    "bash"    49 seconds ago   Exited (0) 37 seconds ago             gallant_hawking
```

To give a new container something to do before it exits, we can include a command like `ls` to list the contents of the default working directory:

```python
$ docker run debian ls

bin
boot
dev
etc
home
lib
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

Once again, we can verify a new container was started and then stopped after the `ls` command was executed:

```python
$ docker ps -a

CONTAINER ID   IMAGE     COMMAND   CREATED              STATUS                          PORTS     NAMES
4cd7aec68643   debian    "ls"      38 seconds ago       Exited (0) 36 seconds ago                 laughing_mcnulty
16223809b3db   debian    "bash"    About a minute ago   Exited (0) About a minute ago             gallant_hawking
```

In some cases, you’ll want to interact with a container and have it start you in a particular working directory. You can achieve this with `docker run` plus the `-it` and `-w` flags:

```python
$ docker run -it -w /home debian

root@5d488c949415:/home#
```

Since the container was started in interactive mode, use `exit` to leave it:

```python
root@5d488c949415:/home# exit
```

Once again, use the `docker ps` command to list all containers:

```python
$ docker ps -a

CONTAINER ID   IMAGE     COMMAND   CREATED              STATUS                      PORTS     NAMES
5d488c949415   debian    "bash"    About a minute ago   Exited (0) 26 seconds ago             magical_shirley
4cd7aec68643   debian    "ls"      2 minutes ago        Exited (0) 2 minutes ago              laughing_mcnulty
16223809b3db   debian    "bash"    3 minutes ago        Exited (0) 3 minutes ago              gallant_hawking
```

A stopped container can be restarted using the `docker container start` command and enough characters of the container ID for Docker to know which container you mean; the `-i` flag restarts the container in interactive mode:

```python
$ docker container start -i 5d

root@5d488c949415:/home#
```

Likewise, a running container can be stopped:

```python
$ docker container stop 5d

5d
```

One thing you’ll notice is that Docker takes a conservative approach to cleaning up unused objects like images and containers. When you stop a container, it is not automatically removed unless you use the `--rm` flag when starting the container:

```python
$ docker run -it -w /home --rm debian
```

To manually clean up, you can delete a container:

```python
$ docker container rm 5d

5d
```

Alternatively, you can use the prune command to delete all stopped containers:

```python
$ docker container prune

WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
4cd7aec68643d855b757ba29cf9a2cd9df50853c88c5307e1286c50a45accb75
16223809b3db97013f1939c30213c6dc11144bf8d389522d6c8d90bc66e5416b

Total reclaimed space: 0B
```

The same sort of garbage collection is needed for images. You can list your images using `docker image ls` with the `-a` flag for all:

```python
$ docker image ls -a

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
debian       latest    bf33d9a9dfb7   3 weeks ago   118MB
```

To delete an image not associated with the container bf33d9a9dfb7:

```python
$ docker image rm bf

Untagged: debian:latest
Untagged: debian@sha256:4d6ab716de467aad58e91b1b720f0badd7478847ec7a18f66027d0f8a329a43c
Deleted: sha256:bf33d9a9dfb709a9a24218f6ea55bab3965d0029cdc2ce01214ad1a87040e530
Deleted: sha256:478649b375d177fb986d0b96804929861077c6c15da73a400fc900afc9187900
```

Or you can use `docker image prune` with -a to remove all unused images:

```python
$ docker image prune -a

WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
```

So those are the basics which provided me with enough information to move onto more interesting scenarios such as hosting Flask apps in containers. My [flask-test](https://github.com/exactful/flask-test) repository on GitHub provides a demo app which can be built and hosted within a container in just a few steps:

```python
$ git clone https://github.com/exactful/flask-test.git

Cloning into 'flask-test'...
remote: Enumerating objects: 24, done.
remote: Counting objects: 100% (24/24), done.
remote: Compressing objects: 100% (21/21), done.
remote: Total 24 (delta 10), reused 5 (delta 0), pack-reused 0
Receiving objects: 100% (24/24), 4.69 KiB | 1.17 MiB/s, done.
Resolving deltas: 100% (10/10), done.
```

The `Dockerfile` contained within the newly-cloned flask-test directory contains all of the information that Docker needs to build an image:

```python
$ cat flask-task/Dockerfile

FROM python:3.8-slim-buster
WORKDIR /usr/src/app
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
#Server will reload itself on file changes if in dev mode
ENV FLASK_ENV=development
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

The image can be built using `docker image build` and the `-t` flag to provide it with a tag of `flask-test-image`:

```python
$ docker image build -t flask-test-image flask-test

Sending build context to Docker daemon   68.1kB
Step 1/9 : FROM python:3.8-slim-buster
3.8-slim-buster: Pulling from library/python
02b45931a436: Pull complete
1b907b3f2a40: Pull complete
ade9c40131e5: Pull complete
6380b003fa8e: Pull complete
1e31a0f45c23: Pull complete
Digest: sha256:9e3036f6b032794efb662f3c579c4c35d0b678bc793590e3e2e217cb5bf1e11b
Status: Downloaded newer image for python:3.8-slim-buster
 ---> db1e59c73041
Step 2/9 : WORKDIR /usr/src/app
 ---> Running in f9c49a06c8df
Removing intermediate container f9c49a06c8df
 ---> c79c8f1f9c1c
Step 3/9 : ENV FLASK_APP=app.py
 ---> Running in 76801af3fdb2
Removing intermediate container 76801af3fdb2
 ---> 4eaf8967484d
Step 4/9 : ENV FLASK_RUN_HOST=0.0.0.0
 ---> Running in 63fb31ebe1d3
Removing intermediate container 63fb31ebe1d3
 ---> 4ff75116a97d
Step 5/9 : ENV FLASK_ENV=development
 ---> Running in bbf385cb1f05
Removing intermediate container bbf385cb1f05
 ---> 97f2f549a0e5
Step 6/9 : COPY requirements.txt requirements.txt
 ---> cb0cc984e762
Step 7/9 : RUN pip install -r requirements.txt
 ---> Running in d3062b512e5e
Collecting flask
  Downloading Flask-2.0.2-py3-none-any.whl (95 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting Jinja2>=3.0
  Downloading Jinja2-3.0.2-py3-none-any.whl (133 kB)
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.0.2-py3-none-any.whl (288 kB)
Collecting click>=7.1.2
  Downloading click-8.0.3-py3-none-any.whl (97 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.0.1-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (27 kB)
Installing collected packages: MarkupSafe, Werkzeug, Jinja2, itsdangerous, click, flask
Successfully installed Jinja2-3.0.2 MarkupSafe-2.0.1 Werkzeug-2.0.2 click-8.0.3 flask-2.0.2 itsdangerous-2.0.1
Removing intermediate container d3062b512e5e
 ---> c0810573e7c5
Step 8/9 : COPY . .
 ---> 91e02222a41b
Step 9/9 : CMD ["flask", "run"]
 ---> Running in df1610a6af6a
Removing intermediate container df1610a6af6a
 ---> 6d7816d54b60
Successfully built 6d7816d54b60
Successfully tagged flask-test-image:latest
```

The now familiar `docker run` can be used to create a container; the `-d` flag runs the container in the background (aka [detached mode](https://www.freecodecamp.org/news/docker-detached-mode-explained/)) and the `-p` flag maps the container’s port 5000 to the Docker host's port 80:

```python
$ docker run -d -p 80:5000 flask-test-image

ae30c4f1e0871f23c91fb85e33501a00ae5dd699f6d46ddc7b9aad3ef74a9e70
```

With Docker now listening on port 80, you can browse to the IP address of your Docker host to see the demo app.  My Raspberry Pi is connected to my local network on 192.168.4.64 so here's what I see on port 80 at http://192.168.4.64:

<figure>
  <img src="/assets/images/blog/flask-app.png" alt="Hello World using Flask and Docker">
  <figcaption>Hello World using Flask and Docker.</figcaption>
</figure> 

I hope this post helps you progress further if you’re just starting out with Docker. From here, I highly recommend [creating services]() using `docker stack deploy` and [persisting container data](https://docs.docker.com/storage/) using volumes.