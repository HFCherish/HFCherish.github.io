---
title: optional with multiple exceptions
toc: true
date: 2019-05-13 11:15:26
tags:
	- java
---

# Optional - something might not be there

Optional` 主要用于避免 null check，但并非消除 null check。不是所有的地方都应该用 Optional。

>`Optional` is not really intended for the purpose of dealing with exceptions, it was intended to deal with potential nulls without breaking the flow of your program. For example:

```java
 myOptional.map(Integer::parseInt).orElseThrow(() -> new RuntimeException("No data!");
```

参见 

* [26 rationals for using optional](https://dzone.com/articles/using-optional-correctly-is-not-optional)
* [the great talk by Stuart Marks](https://www.youtube.com/watch?v=Ej0sss6cq14)

# something might not be there for many reasons

**Concern**：Optional 并不是为了处理所有的 exception，而一个方法没有成功的返回值，可能有很多原因，有时候需要将这个原因暴露个 caller. 此时单纯的使用 Optional 是不合适的。

## checked & unchecked exceptions

此外，对于 exception，我们需要了解 [checked & unchecked exceptions](https://medium.freecodecamp.org/why-you-should-ignore-exceptions-in-java-and-how-to-do-it-correctly-8e95e5775e58) 这两个概念。

1. **checked exceptions**：在 java 中就是非继承了 `RuntimeException` 的异常，表明在代码中必须要处理这种异常。一般用于在出现异常时，必须提供另一种处理途径的场景。
2. **unchecked exceptions**: 在 java 中就是继承了 `RuntimeException` 的异常，不强制要求处理异常。当出现异常时，确实就是要直接终止程序。

![checked & unchecked images](https://cdn-images-1.medium.com/max/1600/1*IxcmkFjjXYQS8KncH7-ChQ.png)

对于前述 concern，checked exceptions 在 java8 Stream API 中是不允许抛出的，而单纯滥用 unchecked exception 则可能会丢失一些必要的处理。

因此解决方案我考虑有两种：

## 封装 Optional 和 exceptions

[这篇博客](https://dzone.com/articles/exceptional-optional) 创建了一个 `Exceptional` 类，当成功时，可以返回 value，失败时可以获取 exception。

这种做法类似于 scala 中的 [Try](https://www.scala-lang.org/api/2.12.8/scala/util/Try.html)。scala 中因为都是 unchecked exceptions，所以提供了 `Try` 来封装必须要处理的异常。在 [scala best practices: do not throw exceptions](https://nrinaudo.github.io/scala-best-practices/referential_transparency/avoid_throwing_exceptions.html) 中指出，不应该抛出任何异常。

不过这种写法，更类似于封装了处理异常的 try catch 写法，以一种可能更简洁的方式来做。

## 不用 Optional

在上述 concern 中，此时不一定需要用 Optional，用了反而不伦不类。