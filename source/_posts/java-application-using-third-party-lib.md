---
title: java.lang.UnsatisfiedLinkError: no xxx in java.library.path
toc: true
date: 2018-08-14 10:07:06
tags:
	- jvm
	- problems
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

