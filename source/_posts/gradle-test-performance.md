---
title: gradle test performance
toc: true
date: 2018-10-12 10:41:39
tags:
	- test
---

[gradle test configurations](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.testing.Test.html#org.gradle.api.tasks.testing.Test:forkEvery)

[one sample config](https://discuss.gradle.org/t/parallel-test-execution-with-gradle-maxparallelforks-property/15136)

[ways to improve performance of gradle build](https://guides.gradle.org/performance/)



common used properties:

* `jvmArgs`: jvm 参数。通常会配置堆栈大小，保证测试对内存的要求。
  * `'-Xms128m', '-Xmx1024m', '-XX:MaxMetaspaceSize=128m'`。`-Xms` 是初始堆大小，`-Xmx` 是最大堆大小，`-XX:MaxMetaspaceSize` 是 class metadata 可占用的最大本地内存（默认是 unlimited）。具体 jvm 参数参考 [java doc](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html). 
* `forkEvery`: 每个 test process 里跑的 test classes 的最大个数。当次数达到限制后，会自动重启。这定义了一个测试线程什么时候回重启，与并发无关。默认是 0，即无最大限制，就是可以一直跑
* `maxParalleForks`: 能并发跑的最大 test processes 数目
* `systemProperty`: 系统属性
* `environment`：系统环境变量
* `include`: 具体执行的测试。可以通过这个配置不同的测试级别（单元测试、集成测试、functional 测试……）





