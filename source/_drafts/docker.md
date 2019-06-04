---
title: docker
toc: true
date: 2019-03-15 16:17:51
tags:
	- container
	- docker
---

# port vs expose

[stackoverflow](https://stackoverflow.com/questions/40801772/what-is-the-difference-between-docker-compose-ports-vs-expose)

## port

define the mapping between host port & container port, so that others can access the container from outside of docker.

* **Access-Outside-Of-Container**

* **Between-Docker-Networks-And-Host-Machine**

You can define multiple mappings.

And you can omit the `hostport`, on which case hostport will be generated automatically.

```yaml
ports:
	- [HOST_PORT]:[CONTAINER_PORT] 
	- [CONTAINER_PORT]
	- [HOST_PORT2]:[CONTAINER_PORT]
```

## expose

define the container port, so that other containers in docker can access it.

* **Access-Between-Containers**

* **In-Docker-Networks**

You can only define a single value for `expose`

