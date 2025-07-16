---
layout: post
title: Notes - Docker custom images
description: Notes from online course "Docker and Kubernetes - The Complete Guide"
published: True
---

### Docker file template
1. Use existing docker image as a base
2. Download and install dependencies
3. Tell image what to do when it starts

Example:
```Dockerfile
FROM alpine

RUN apt add --update redis

CMD ["redis-server"]
```

- **base image** - set of commands, tools that we need to run our app, like `apk`

### Building image
- by default if looks for `Dockerfile` in the current folder

```shell
docker build .
```

> [!Info]
> Each build start with a context which is a set of files and folders we want to use in our build.

- For each step there is an **temporary container** where the action is running, we can see it in build log output.

Step 2 is using image `234132bh24b3` and after the `RUN` instruction a new image is created with `jh345jh3b3k` the result of `RUN` instruction.

```shell
...
Step 2/3 : RUN apt add --update redis
---> Running in 234132bh24b3
fetch http://...
(1/1) Installing redis
Executing redis
OK: 6Mib in 14 packages
Removing intermediate container 234132bh24b3
---> jh345jh3b3k
...
```

### Flow of events
1. FROM ...
	- download image or check local cache
2. RUN ....
	- get image from previous step
	- create a container out of it
	- run the command in the container
	- take a snapshot of the container
	- shutdown temporary container
3. CMD
	- get image from previous step
	- create a container out of it
	- tell container what it should do when started
	- create container with modified primary command
	- shutdown temporary container
 4. Output is image from previous step

### Tagging an image
- so far we were using IDs with our custom images

```
docker build -t mytag .
```

- convention for tags `<your docker ID>/<project or repo name>:<version>`
	- technically only `version` is the tag, the stuff before is the name

### Manual image generation with Docker Commit
- you can create a new image by doing `commit` by specifying run command from a running container

```
# run alpine image and install Redis
docker run -it alpine sh
# in contianer install Redis
apt-get install redis

# in a new tab, get ID of running container
docker ps
123123

# create a new image by doing commit, and overwrite CMD
docker commit -c 'CMD ["redis-server"]' 123123
sha256:qweq123vwernweuifnf

# run the new image
docker run qweq123vw
```
