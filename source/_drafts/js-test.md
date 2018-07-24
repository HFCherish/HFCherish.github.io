---
title: js test
toc: true
date: 2018-06-15 19:04:56
tags:
	- javascript
---

[前端测试框架对比](https://www.cnblogs.com/lihuanqing/p/8533552.html) 介绍了现在主流的测试框架。我重点学一下 jest，开箱即用，配置少，入门最佳（不行的时候再学 mocha，类似，只不过多了些配置）

jest 特性：

* 提供测试框架(Mocha, Jasmine, Jest, Cucumber)
* 提供断言(Chai, Jasmine, Jest, Unexpected)
* 生成，展示测试结果(Mocha, Jasmine, Jest, Karma)
* 快照测试(Jest, Ava)：测试 UI 或数据结构是否和之前完全一致，通常 UI 测试不在单元测试中
* 提供仿真(Sinon, Jasmine, enzyme, Jest, testdouble)。仿真即 mocks, spies, and stubs
* 生成测试覆盖率报告(Istanbul, Jest, Blanket)
* 提供类浏览器环境(Protractor, Nightwatch, Phantom, Casper)

# [snapshot test](https://facebook.github.io/jest/docs/en/snapshot-testing.html)

# [api](https://facebook.github.io/jest/docs/en/api.html)