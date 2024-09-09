# Docker CLI Commands

- [Docker CLI Commands](#docker-cli-commands)
  - [Install](#install)
  - [General](#general)
  - [Images](#images)
  - [Containers](#containers)
  - [Loggins and System](#loggins-and-system)
  - [Docker Compose](#docker-compose)

---
## Install

* Linux/Ubuntu: Follow the steps in 
https://docs.docker.com/engine/install/ubuntu/

## General

* `docker --help`: Get help with Docker

* `docker info`: Display system-wide information

## Images

* `docker images`: List images in local system
 
* `docker build -t [image_name]`: Build an Image from a Dockerfile (inside a
directory)

* `docker rmi [image_name/id]`: Delete Image `[image_name/id]`

* `docker image prune`: Remove all unused images

## Containers

* `docker ps`: List running containers. Add argument `-a` to list all the
containers

* `docker container stats`: View resource usage stats

* `docker run --name [container_name] [image_name]`: Create and run a container 
from an [image_name], with a [container_name]

* `docker run -p [host_port]:[container_port] [image_name]`: Run a container
and publish its port(s) to the host

* `docker start|stop [container_name/id]`: Start or stop an existing container

* `docker rm [container_name]`: Remove a stopped container

## Loggins and System

* `docker logs -f [container_name]` Fetch and follow the logs of a container
  
* `docker inspect [container_name]`: Inspect a running container

* `docker top [container_name]`: List the processes running inside a container

* `docker system prune -a`: Remove all unused containers, networks, images 
(both dangling and unused), and volumes

## Docker Compose

In the directory where the `docker-compose.yml` file is located:

* `docker compose build`: (Re)Build instructions in a `docker-compose.yml` 
file

* `docker compose up`: Build (if not), (re)create, start, and attach to 
containers for a service

* `docker compose down`: Stop containers and remove containers, networks, 
volumes, and images created by `up`

* `docker compose start`: Start existing containers for a service

* `docker compose stop`: Stop running containers without removing them

* `docker compose logs`: Monitor the output of your running containers and 
debug issues

* `docker compose ps`: List all the services along with their current status