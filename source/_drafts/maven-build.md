---
title: maven build
toc: true
date: 2019-03-29 14:51:44
tags:
	- maven
	- build
---

# inheritance vs multimodule

[post](https://blog.csdn.net/isea533/article/details/73744497)

**The `packaging` type required to be `pom` for *parent* and *aggregation* (multi-module) projects**. These types define the goals bound to a set of lifecycle stages. For example, if packaging is `jar`, then the `package` phase will execute the `jar:jar` goal. Now we may add values to the parent POM, which will be inherited by its children. Most elements from the parent POM are inherited by its children, including:

## package type: pom

[stackoverflow](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven)

**`pom` is basically a container of submodules**, each submodule is represented by a subdirectory in the same directory as `pom.xml` with `pom` packaging.

Somewhere, nested within the project structure you will find artifacts (modules) with `war` packaging. Maven generally builds everything into `/target` subdirectories of each module. So after `mvn install` look into `target` subdirectory in a module with `war` packaging.

# scope

[post](https://www.jianshu.com/p/a4fc54b5a6bf)

