---
title: test principles
toc: true
date: 2018-10-12 13:49:16
tags:
	- test
---

[three reasons why we should not use inheritance in tests](https://www.petrikainulainen.net/programming/unit-testing/3-reasons-why-we-should-not-use-inheritance-in-our-tests/)

大概意思是：

1. 很多测试里的继承用的不合适。测试也是代码，必须符合继承的原则。

> The point of inheritance is to **take advantage of polymorphic behavior NOT to reuse code**, and people miss that, they see inheritance as a cheap way to add behavior to a class. When I design code I like to think about options. When I inherit, I reduce my options. I am now sub-class of that class and cannot be a sub-class of something else. I have permanently fixed my construction to that of the superclass, and I am at a mercy of the super-class changing APIs. My freedom to change is fixed at compile time.

多态的定义：

> Polymorphism is the provision of a single interface to entities of different types



2. 可能会影响性能。

并不是所有的测试可能都会用到测试基类里的东西，而 beforeClass、beforeTest 在执行所有测试时都会被1. 查找，2. 执行，这些都导致了 CPU 的浪费。

此外，其他人也提到，比如 springboottest 中，基类中配置 `@SpringBootTest` ，最终可能导致在每个测试中配置 `@DirtyContext`等，严重影响性能。

3. 很影响 reading

测试即文档，而很多东西却是需要读多个类



这个问题仁者见仁，智者见智。duplicate code 也是问题。两者相权，如何处理？但对于目前的代码，如果没有问题，还是保持现状比较好。对于符合继承规则这点，倒是需要注意的。



作者在 [writing clean test](https://www.petrikainulainen.net/programming/testing/writing-clean-tests-it-starts-from-the-configuration/) 中提出了些解决方法，但感觉还是有些片面。当然他写了一个系列的文章：[writing clean test series](https://www.petrikainulainen.net/writing-clean-tests/)。



另外读书：[effective unit testing](https://www.amazon.cn/dp/1935182579/ref=sr_1_2?ie=UTF8&qid=1539322654&sr=8-2&keywords=Effective+Unit+Testing%3A+A+guide+for+Java+developers)

