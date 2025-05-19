---
layout: post
title: Notes - Docker basics
description: Notes from online course "Docker and Kubernetes - The Complete Guide"
published: True
---

## Basic commands
### run

```bash
docker run <image name> command
```

- with command you can overwrite the default behaviour

```bash
docker run busybox echo hi there
docker run busybox ls
docker run -it busybox sh
docker run busybox ping www.google.com
```

> [!Busybox]
> Lightweight image with several Unix utilities.
> Usage:
> - utility OS - stripped down version of many common linux utilities
> - common usage - used for basic tasks of base image for more complex apps
> - creating containers - it has basics structure so good to expand on
> - interactive session - to run shell inside the container `docker run -it busybox sh`

### ps
- list running containers

```bash
# all containers ever created
docker ps --all
```

### create
- create the files system structure of the image

```bash
docker create hello-world
# spits out ID of the create container
912314234...
```

### start
- run the startup command

```bash
docker start -a 912314234...
# -a : attach to contianer and print out the output
```

- once the container is finished you can still list it with `docker ps` command
- if you take the ID from `docker ps`, you can start the same command again

```bash
docker ps --all
sdd123asd ...

docker start -a sdd123asd
```

- with `docker start` you cannot replace the default program

```bash
docker start -a sdd123asd /bin/bash
# throws error, the command was already set for image sdd123asd
```

### system prune
- removes stopped containers, networks, images, cache

### logs
- get logs from a container

```bash
docker create busybox echo hi
# asd871982hed9

docker start asd871982hed9
# hi

docker logs asd871982hed9
# hi
```

### stop
- stop the container
- sends `SIGTERM` message to container
	- gives the process a time to finish and shutdown, so lets the app handle shutdown
- if does not stop in 10sec then `SIGKILL` is send

```bash
docker create busybox ping www.google.com
# asd1231
docker start asd1231
docker logs asd1231
# ping ...

docker stop asd1231
```

### kill
- stops the container
- sends `SIGKILL` message
- just kills container

### exec
- executes additional command in the container
- `-it` argument allows us to provide input to the container
	- `-i` attach terminal to standard input / interactive
	- `-t` very simple explanation is that it makes sure that input and output is better formatted

```bash
docker exec -it CONTAINERID redis-cli
```

- example is with running `Redis` in container and then running `redis-cli` to access it.

```bash
docker run redis
docker ps
# get ID of running container
docker exec -it asdaf123 redis-cli
```

- typical use is to run terminal inside the container

```bash
docker exec -it asdf123 /bin/bash
```
