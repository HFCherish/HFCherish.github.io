---
title: java.lang.UnsatisfiedLinkError: no xxx in java.library.path
toc: true
date: 2018-08-14 10:07:06
tags:
	- jvm
	- problems
	- ini
---

# 背景

1. 项目需要引入 local 第三方包
2. 该第三方包只有 window/linux license，而开发在 macos
3. 开发时，通过 gradle dependency `compile files('path/to/thejar.jar')` 来引入包

# 问题

运行时，报错误 `java.lang.UnsatisfiedLinkError: no thejar in java.library.path`

# 原因

引入 `.dll` 或 `.so` 失败造成。

# solution

1. 把 thejar 加入到 path 中 ————— not work
2. 加入 path，并 loadLibrary ——————— not work
3. should work（配置 .dll 或 .so 路径）：
   1. 配置 PATH
   2. 或 jar 包启动时，设置 '-Djava.library.path'



有关 `PATH`, `-classpath`, `java.library.path` 的区别，再 google。java 在使用这三个 path 时：

1. `PATH`：用来寻找 `java`, `javac` 等 command 并执行
2. `classpath`：jvm 在执行时用来寻找 java class。classpath 一般指向 jar 包的位置，即 jdk 的 lib 目录
3. `java.library.path`: 指向非 java 类包的位置，如 .dll, .so 等。在 java `System.loadLibrary()` 时从这找



`java.library.path` 的设定可参照 [stackoverflow](https://stackoverflow.com/questions/1403788/java-lang-unsatisfiedlinkerror-no-dll-in-java-library-path)

* `java -jar xxx.jar -Djava.library.path=xxx`
* Set PATH
* Set programmatically (NOT RECOMMENDED, may cause unpredictable result):

```java
System.setProperty("java.library.path", path);
//set sys_paths to null
final Field sysPathsField = ClassLoader.class.getDeclaredField("sys_paths");
sysPathsField.setAccessible(true);
sysPathsField.set(null, null);
```



linux 下可通过 `LD_LIBRARY_PATH` 设置 `java.library.path`。windows 下似乎是通过 `PATH` 设定（待确定）

