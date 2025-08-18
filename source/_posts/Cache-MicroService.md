---
title: Cache - MicroService
toc: true
tags:
  - cache
date: 2020-06-03 10:42:53
---

# Where is my cache for a service

[Architectural Patterns for Caching Microservices](Architectural Patterns for Caching Microservices)

Patterns:

1. **embedded**: save cache in the service
2. **client-server**: a completely separate cache server
3. **reverse-proxy**: put the cache in front of each service
4. **Sidecar**: put the cache as a sidecar container that belongs to the service

How does cache work? The application receives the request and checks if **the same request** was already executed (and stored in the cache)

## Embedded

**Embedded Distributed Cache**

Why distributed?

1. Same requests happen on different instances, and we want to use cache on the second same request no matter whether they reached the same instance.
2. An update request reached one instance, and we want all the cache of this resource on all instances should be updated.

## Client-Server

pros:

* sperarate server, so you can use any programming language you want
* separate management. you can scale up/down, do backup, design security separatly on need.

cons:

* a new deployment & related ops work (cloud can make this simple)

## Sidecar

A mixture of Embedded & client-server. ([sidecar container pattern](https://hazelcast.com/blog/hazelcast-sidecar-container-pattern/))

Take k8s as example (now most of the sidecar pattern is implemented on k8s, becaused it supports this pattern inborn), we put the cache server with the service as separate containers on the same pod. The request reach the service container first, and then the service can access the cache server by `localhost` .

> The [sidecar pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar) is a technique of attaching an additional container to the main parent container so that both would share the same lifecycle and the same resources. You may think of it as a perfect tool for decomposing your application into reusable modules, in which each part is written in a different technology or programming language.

## Reverse proxy

1. Everytime you try to access the servcie, you go to the reverse proxy (e.g. nginx). 
2. And the proxy will first check the cache for the request. If cache exists, return immediately, else, forward to the service and write the cache.

pros: 

* In this way, the service has no idea about the cache, so nothing will change on the service when cache introduced.

cons:

* cannot use any application-based code to invalidate the cache.

Also, you can put the reverse proxy into the side-car, that is, **reverse-proxy-sidecar** pattern.

Now, there is no mature **HTTP Reverse Proxy Cache Sidecar** solution at all. Nginx can do it, but it's not a good choice since it's not mature.

# Caching Practices

* Always use caching in one place for a service. Mutliple caches will make the cache invalication and error-prone difficult.
