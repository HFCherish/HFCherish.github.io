---
title: gradle basic
toc: true
date: 2018-09-10 19:15:21
tags:
	- gradle
---

gradle 不同于 maven 的：

1. maven 基于 xml，而 gradle 基于 groovy。`build.gradle` 写的全是 groovy 代码，是一种 script。所以它的可塑性很大，实现的东西很多。而 xml 只是一个配置，并不是代码，实际的运行是依靠 maven daemon 来基于配置执行的。
2. 



# gradle in intellij

[gradle in intellij video](https://www.youtube.com/watch?v=JwPYjnhah3g)

1. 可以在 intellij 中配置 build、test 等使用 gradle runner

![using gradle runner in intellij.png](https://upload-images.jianshu.io/upload_images/721960-c84ba6346d785665.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 在 `build.gradle` 中可以 `command + n` 添加 dependency
3. 运行 gradle 测试可以通过 gradle window，找到相应的 task 执行，它会显示在 intellij 的测试运行窗口。（同样，每当添加了插件后，都可以在这看到支持的 tasks）

![gradle test in intellij.png](https://upload-images.jianshu.io/upload_images/721960-a97b69ebc8a99cbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)