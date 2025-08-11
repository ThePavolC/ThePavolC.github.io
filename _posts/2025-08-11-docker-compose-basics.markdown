---
layout: post
title: Notes - Docker Compose Basics
description: Notes from online course "Docker and Kubernetes - The Complete Guide"
published: True
---

## Docker Compose
- combines the most common actions used with Docker into an easy to use tool

### Sample docker-compose file

```docker
version: '3'
services:
  redis-server:
    image: 'redis:latest'

  my-web-app:
    build: .
    restart: always
    ports:
      - '4001:8081'
    depends_on:
      - redis-server
```

#### Benefits
- services are automatically on the same network
	- services can communicate using service name (eg. `redis-server`)
- simpler Dockerfile with ports config in a single place
- easy orchestration of multiple services
- grouped commands and easy tools

#### Commands

##### docker-compose up
- combined build and run into one
- using docker-compose file
- add `--build` to rebuild images if there was some change
- add `-d` to start in services in the background
##### docker-compose down
- stop all services defined in the docker-compose.yml in the current folder
##### docker-compose ps
- `ps` view of services in the docker-compose.yml file

#### Restart policy
- `"no"` - never restart
	- it has to be in quotes because in YAML `no` is treated as boolean so you need "no"
- `always` - restart for any reason, even if success
- `on-failure` - only restart when an error code
- `unless-stopped` - always restart unless we (the developers) forcibly stop it
