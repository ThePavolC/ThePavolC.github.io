---
layout: post
title: Notes - Docker Production Workflow
description: Notes from online course "Docker and Kubernetes - The Complete Guide"
published: True
---

# Source code flow
- `feature` branch -> Pull Request -> `main` branch -> run tests -> CI/CD -> AWS Hosting

# Workflow
- have separate Dockerfile for dev and for prod

```shell
docker build -f Dockerfile.dev .
> sha:123123

# start the built container
docker run 123123

# to open ports
docker run -p 3000:3000 123123
```

- to develop, you need to run build, run, then do changes, and do same again
	- to not do all that you can use `volumes`

### Volumes
- with volumes we set up reference to point to the files or folders on the local machine

```shell
# -v /app/node_modules - put a bookmark on the node_modules folder
# -v $(pwd):/app - map the working folder into the /app folder
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <img-ID>
```

- "bookmarking volumes"
	- `-v /app/node_modules` - we did not map it to a local folder
	- we want this folder to be a placeholder and not map it against the local folder, so it's not mounting it

> In my Mac OS instance, I was not getting updates when running react project in docker and with mounted src volume.
> I had to use env variable `WATCHPACK_POLLING=true` which then synced files but with about 60 sec delay.

- with volumes, do we need `COPY . .`?
	- technically we do not need it but we are only using docker-compose to mount volumes, in case that changes in the future then we still need `COPY` instruction

- to simplify volumes mounting, use docker-compose

```docker-compose.yml
version: '3.8'
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - WATCHPACK_POLLING=true
```

### Tests
- we can just run tests on the build image

```shell
docker build -f Dockerfile.dev .
> sha:123123

docker run -it 123123 npm run test
```

Other ways to run tests:

1. Attach to the running container

```shell
docker ps # get container ID
docker exec -it 123123 num run test # run tests on the running container
```

2. Use docker-compose

```docker-compose.yml
  tests:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]
    environment:
      - WATCHPACK_POLLING=true
```

- with this approach we are only watching logs but we can't interact with the tests
	- could we use `docker attach` ?
		- when we run `npm run test`, node will start a process with the tests so when we attach to the image, we attach to the primary process of 1 which is node command and not the test process
		- so, it will no work

## Nginx
- separate instance of web server to serve our production files
- we want to build a production code in one step and then use nginx to serve it

```Dockerfile
FROM node:lts-alpine AS builder

WORKDIR '/app'

COPY package.json .
RUN npm install

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html
```

- then simply build and run

```shell
docker build . # since prod stuff will be in Dockerfile
docker run 123123
```
