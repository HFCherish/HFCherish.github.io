---
title: docker
toc: true
date: 2019-03-15 16:17:51
tags:
    - container
---

# container

## what

## live where

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

# References

1. Kubernetes for the Absolute Beginners - Full Course (FreeCodeCamp.org) - This beginner-friendly video course provides a hands-on introduction to Kubernetes. It covers core concepts, deploying applications, scaling, and more. [Link to Course](https://www.youtube.com/watch?v=ZebWSI8KYfY)

2. Kubernetes Fundamentals (Official Kubernetes YouTube Channel) - This series of short videos by the official Kubernetes YouTube channel covers the fundamentals of Kubernetes, including architecture, concepts, and deployment techniques. [Link to Playlist](https://www.youtube.com/playlist?list=PLbG4OyfwIxjEcgdi9zS_YTPnniferR_Ot)

3. Docker and Kubernetes: The Big Picture (Pluralsight course) - This course by Nigel Poulton provides a high-level overview of Docker and Kubernetes, exploring their key features, use cases, and how they work together. [Link to Course](https://www.pluralsight.com/courses/docker-kubernetes-big-picture)

4. Kubernetes Essentials (edX course) - This course offered by the Linux Foundation on the edX platform covers the essentials of Kubernetes, including deployment, scaling, and managing applications. [Link to Course](https://www.edx.org/professional-certificate/kubernetes-essentials)

5. Docker Mastery: The Complete Toolset From a Docker Captain (Udemy course) - This highly regarded video course by Bret Fisher provides a comprehensive understanding of Docker and its ecosystem. It covers containerization, Docker Compose, Docker Swarm, and deploying applications. It can be completed within a few days with focused study. [Link to Course](https://www.udemy.com/course/docker-mastery/)

6. Kubernetes Basics (Official Kubernetes Documentation) - The official Kubernetes documentation provides a wealth of information and tutorials to learn Kubernetes. The "Kubernetes Basics" section offers a step-by-step guide to deploying and managing applications on Kubernetes. It is well-documented and provides practical examples. [Link to Documentation](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

7. Terraform Getting Started (HashiCorp Learn) - HashiCorp Learn offers a "Terraform Getting Started" guide that covers the fundamentals of Terraform. It provides hands-on exercises and explanations of key concepts such as infrastructure as code and resource management. Completing this guide will give you a solid foundation in Terraform. [Link to Guide](https://learn.hashicorp.com/tutorials/terraform/getting-started-build)

8. Kubernetes in 10 Minutes (Official Kubernetes YouTube Channel) - This quick video tutorial by the official Kubernetes YouTube channel provides a concise introduction to Kubernetes, explaining key concepts in just 10 minutes. It's a great starting point to grasp the basic concepts of Kubernetes before diving deeper. [Link to Video](https://www.youtube.com/watch?v=JXhqp2_XeUQ)

9. Terraform Crash Course for Absolute Beginners (FreeCodeCamp.org) - This video tutorial provides a crash course on Terraform, covering its key concepts and demonstrating how to provision infrastructure using code. It's a beginner-friendly resource that can help you quickly get up to speed with Terraform. [Link to Video](https://www.youtube.com/watch?v=SLB_c_ayRMo)

1. Docker Documentation - The official Docker documentation is a comprehensive and well-documented resource that covers all aspects of Docker. It includes tutorials, guides, and references to help you learn Docker from the ground up. [Link to Documentation](https://docs.docker.com/)

2. Kubernetes Documentation - The official Kubernetes documentation provides extensive information on Kubernetes architecture, concepts, and usage. It includes tutorials, examples, and guides to help you understand and work with Kubernetes. [Link to Documentation](https://kubernetes.io/docs/home/)

3. Terraform Documentation - The official Terraform documentation offers in-depth explanations, tutorials, and examples for learning Terraform. It covers topics such as infrastructure provisioning, resource management, and Terraform modules. [Link to Documentation](https://www.terraform.io/docs/index.html)

4. Katacoda - Katacoda provides interactive, browser-based learning environments for various technologies, including Docker, Kubernetes, and Terraform. They offer free hands-on scenarios and tutorials that allow you to practice and learn in a sandboxed environment. [Link to Katacoda](https://www.katacoda.com/)

5. YouTube Channels - There are several YouTube channels that offer free tutorials and explanations on Docker, Kubernetes, and Terraform. Some recommended channels include:
   
   - TechWorld with Nana: Provides tutorials on Docker and Kubernetes. [Link to Channel](https://www.youtube.com/c/TechWorldwithNana)
   - TechWorld with Nana - DevOpsLab: Focuses on DevOps-related topics, including Docker and Kubernetes. [Link to Channel](https://www.youtube.com/c/TechWorldwithNanaDevOpsLab)
   - HashiCorp: Offers official tutorials and guides for Terraform. [Link to Channel](https://www.youtube.com/user/HashiCorpVideos)
