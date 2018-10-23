---
title: performance for io in java
toc: true
date: 2018-10-15 12:59:58
tags:
	- java
---



# java.io.ByteArrayOutputStream

这一般在用到字节流是会用到。

[java performance tuning guide](http://java-performance.info/java-io-bytearrayoutputstream/) 这篇文章不建议在 performance-criticted 代码中使用 `ByteArrayOutputStream`：

1. **同步写入，效率低**

> `ByteArrayOutputStream` allows you to write anything to an internal expandable byte array and use that array as a single piece of output afterwards. Default buffer size is 32 bytes, so if you expect to write something longer, provide an explicit buffer size in the `ByteArrayOutputStream(int)` constructor
>
>  注：
>
> 1. `ByteArrayOutputStream` 内部是一个可变长度的 byte[]（通过扩充实现可变）。它有个初始长度（默认 32），可以在 constructor 中指定.
> 2. `ByteArrayOutputStream` 是同步写入，比较影响效率

2. **toByteArray() 效率低，使用 toString(charset)**

`toByteArray` 执行了一遍拷贝，效率低。`toString` 则使用 `String(bytes[])` 直接将内部 byte 转成了 String

[inefficient byte[] to String constructor](http://java-performance.info/inefficient-byte-to-string-constructor/) 指出 `String(bytes[])`  也是 copy，但 java8 中的源码看了下，似乎不是 copy，待考证……