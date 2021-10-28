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

## chmod & echo

```dockerfile
RUN chmod -R a+x /tmp/dir/
RUN echo $(ls -l /tmp/dir)
RUN FILE="$(ls -l /tmp/dir)" && echo $FILE
```

如果 `echo` 没有在 `docker build` 时打印出来，则可以通过指定 `docker build --progress=plain` 参数来显示（没有打出来也可能是因为 cache 的原因，也可以 no-cache 指定一下） 
