---
title: k8s general
toc: true
tags:
  - container
date: 2019-03-01 15:41:12
---

[k8s](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/) is a platform to manage containerized workloads and services.

# Concepts

## kubernetes Objects

1. Kubernetes abstract **a desired state of cluster** as objects.
2. an object configuration includes:
   1. spec: describe the desired state
      1. apiVersion: the api version of kubernetes
      2. metadata: the name & namespace
      3. spec: the desired state definition.
   2. status: describe the actual state of the object
3. cluster state ([understanding kubernetes objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)):
   1. what containerized applications are running and where they're running
   2. how many resources (disk, network, etc.) are attached to the the container
   3. the policies around how the application behaves, such as restart policies, fault-tolerance, etc.

### Workloads

Objects that set deployment rules of pods.

All controllers are workloads.

### POD

[pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) is **a minimize runnable object** in k8s object model.

A Pod encapsulates an application *container* (or, in some cases, multiple containers), *storage resources*, *a unique network IP*, and *options that govern how the container(s) should run.*

It represents **a single instances of application**.

There're two common use cases:

* **Pods that run a single container**, which is the most case. So often pod is a synomynous with container.
* **Pods that run multiple containers that need to work together.** All the containers in a pod share the storage & network, which means they use the same ip and same storage volume.

#### Pod lifecycle

**A pod doesn't self-heal & self-scale**. It's just a running instance. It stopped & deleted when the process is finished, or the node is failed, or the resource is exhaused… Controllers (eg. deployment, statefulSet…) instead can create and manage multiple pods.

Pod restarting is different from container restarting. **Pod provides the env for containers: os, storage, network…**

### Controllers

Controllers (eg. deployment, statefulSet…) instead can create and manage multiple pods.

### StatefulSet????

When using a Kubernetes StatefulSet, each pod in the StatefulSet is typically assigned its own unique and stable hostname and ordinal index. This allows each pod to have its own identity and ensures that the pods maintain a predictable and consistent network identity across restarts or rescheduling.

By default, each pod in a StatefulSet is provisioned with its own volume(s), which can be backed by physical storage or cloud-based storage solutions. This ensures data isolation and allows each pod to have its own storage space.

However, it is possible to configure a StatefulSet to use shared storage among its pods. This can be achieved by using a shared network file system (NFS) or a distributed file system (such as CephFS or GlusterFS) as the underlying storage solution. With this configuration, all pods in the StatefulSet can mount the same shared storage, allowing them to access and share data stored within that storage.

To achieve shared storage with StatefulSet, you would need to configure the appropriate volume and volume mount settings in the StatefulSet's pod template specification. The specific configuration details depend on the storage solution you are using.

It's important to note that while sharing storage among pods can provide benefits like data sharing and consistency, it may introduce potential performance and scalability considerations. It's recommended to carefully evaluate your requirements and choose the appropriate storage solution based on your specific use case.

### Service

Service is an **abstraction** which defines **a logical set of pods**, and **a policy by which to access the pods**.

It enables **the decoupling of access & the real pods**, which means you can access by service name/ip rather than the pod ip, while **pod ip is not stable**.

> Q: How does k8s get all pod endpoints by service?
> 
> A: By `LabelSelector`. You can define labels for each pod. And we define `LabelSelector` in service, so that k8s can search pod node by label first, and then using the `targetPort` to locate the pod on node.

### Secrets???

In Kubernetes (k8s), a secret is an API object used to store sensitive data such as passwords, API keys, and TLS certificates. Secrets are typically used to securely provide this sensitive information to applications running within Kubernetes pods.

Kubernetes supports different types of secrets, each designed for specific use cases. Here are a few examples:

1. Opaque: The most commonly used secret type in Kubernetes. It stores arbitrary key-value pairs as Base64-encoded strings. These secrets are useful for storing generic sensitive data.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: dXNlcm5hbWU=
  password: cGFzc3dvcmQ=
```

2. TLS: Specifically used for storing TLS certificates and private keys. This type of secret is often used for securing HTTPS connections.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <Base64-encoded TLS certificate>
  tls.key: <Base64-encoded private key>
```

3. Docker-registry: Used to store authentication information for private Docker registries.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: reg-cred
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <Base64-encoded Docker registry credentials>
```

These are just a few examples of secret types in Kubernetes. Secrets allow you to securely manage sensitive information and provide it to your applications when needed.

By default, `echo` command adds a newline character at the end of the output. The `-n` option prevents `echo` from appending the newline, ensuring that only the encoded string is printed without any additional characters.

```shell
echo -n "test" | base64
dGVzdA==
```

## kubernetes control plane

1. the control plane **manage the cluster state to match the desired state of objects**
2. the control plane consists of a collection of processes for the above intention.
   1. kubernetes master:
      1. is responsible for the maintaining the desired state.
      2. "master" in fact refers to three processes: [kube-apiserver](https://kubernetes.io/docs/admin/kube-apiserver/), [kube-controller-manager](https://kubernetes.io/docs/admin/kube-controller-manager/) and [kube-scheduler](https://kubernetes.io/docs/admin/kube-scheduler/).
      3. these three processes are typically run on single node in the cluster. This node is called "master", too. The master can also be replicated for avaibility and redundancy.
   2. kubernetes nodes:
      1. the nodes to run applications.
      2. there're two process in each node:
         1. kubelet: communicate with the master
         2. kube-proxy: a network proxy.

# kubernetes api

The **api** communicates with master to operate on kubernetes objects.

**kubectl** is a cli to implement the above intention. It in fact calls the api internally.

All kinds of **sdk** (java, python...) encapsulate the api, too.

There are two kinds of api groups:

1. core groups: the original k8s api
2. other groups: other api to extend the core group, like you may want to abstract more objects?

# Access services on cluster

There are several levels.

ip:

* pod ip: 
* cluster ip: the virtual ip for a service
* node ip

## In-pod access

**Each pod has a unique IP**.  And all **containers in a pod share the ip**, which means that they can access each other by `localhost`

## In-cluster access

By default, a service can be accessed by other pods in the same cluster, through:

#### cluster-ip:port

* cluster ip is the virtual ip of a service
* port is is the node port of the service.

Every node on k8s has a `kube-proxy`. It installs iptable rules which keep a simple record of (servicename:clusterip:serviceport).

In some node:

1. call cluster-ip:port————— **In a pod**
2. the kube-proxy search the iptable, and get some info.———— **pod => kube-proxy of node where the pod resides.**
3. the kube-proxy using the info to ask master for the real endpoints.—— **kube-proxy of node => master kube-api**
4. the kube-proxy chooses a endpoint by `SessionAffinity` defined in service (round-robin by default) —— **in kube-proxy**
5. the kube-proxy redirect request to `pod-ip:podPort` —— **kube-proxy => pod endpoint**

kube-proxy: enables the Kubernetes service abstraction by maintaining network rules on the host and performing connection forwarding

![proxy-mode:user space](https://d33wubrfki0l68.cloudfront.net/e351b830334b8622a700a8da6568cb081c464a9b/13020/images/docs/services-userspace-overview.svg)

#### servicename.namespacename

When accessing by service name, there're two ways:

##### environement viaribles

when create a service, k8s will create some env for each serivce, e.g. `{SVCNAME_CAPTIPAL}_SERVICE_HOST`. So the kube-proxy in fact gets the service cluster ip from these envs first. When a pod calls a service, t**he service must be created before the pod** so that the envs are created.

```properties
# the cluster ip
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

##### DNS

[DNS](https://kubernetes.io/docs/concepts/services-networking/service/#dns) is an add-on k8s object, which can be chosed to add to the cluster.

It's an in-cluster dns, which serves DNS records for Kubernetes services.

It watches the k8s api for new services and create **DNS SVC records** for each.

Containers started by Kubernetes automatically include this DNS server in their DNS searches.

## between-cluster access

1. NodePort: expose nodeIP
2. LoadBalancer: expose loadbalancer ip
3. Ingress: expose loadbalancer & path

# Ingress

[一篇搞懂 ingress](https://zhuanlan.zhihu.com/p/618328589)

## what's ingress

Ingress is a k8s resource to route external traffic into k8x cluster services. From functionality perspective, it provides:

1. load balance











































1. routing

2. reverse proxy

3. service discovery

4. ...

It can route by http path, host name, http method, etc.

## IngressController

However, Ingress itself is just a definition. IngressController is the one to really routing, load-balancing, and so on. IngressController config the load balancer etc. based on the ingress resource. And it routes based on the rules defined in ingress.

IngressController is an independent component. There're many implementations. And different controller has different features, i.e. not all IngressControllers have all the above functionalities.

- nginx ingress controller: in this case, it converts the ingress object innto nginx config, and apply this config to nginx server.

- traefix ingress controller

- istio ingress gateway

- contour ingress controller: based on Envoy proxy.

- kong ingress controller: a kinds of api gateways

- ambassador api gateway: k8s-native api gateway

## expose the service

When external user wants to access some cluser service, the entrypoint is the address of IngressController. To expose IngressController, several ways:

### 1. NodePort

Expose service to all nodes in cluster.

### 2. HostNetwork(共享主机网络)

Expose ingressController to the network namespace of  host node, so that we can access IngressController by the host ip.

### 3. LoadBalancer

Expose IngressController to the LB, and then access through LB

### 4. ExternalIPs

Designate externalIP for ingressController, and access throught that.