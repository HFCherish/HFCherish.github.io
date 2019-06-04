---
title: 构建
toc: true
date: 2018-08-16 11:26:13
tags:
	- gradle
	- build
	- java
---

# Problems?

1. manifest 是干什么用的？
2. 代码运行时，如何找到 dependency 的包
3. java -jar 时，classpath 指定？



# classpath

classpath 指定的是 java 类所在的目录（包括当前项目的类、依赖的类等）。应该是当打 jar 包的时候，默认会加上当前目录(.)到 classpath，这样就包含了 jar 内部的类？



# implementation vs compile vs api

[stackoverflow](https://stackoverflow.com/questions/44493378/whats-the-difference-between-implementation-and-compile-in-gradle)