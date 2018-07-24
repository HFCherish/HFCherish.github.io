---
title: js ecosystem
toc: true
date: 2018-06-21 11:54:38
tags:
	- javascript
---

我在学习 react，一直在使用 [create-react-app](https://github.com/facebook/create-react-app) 创建项目。create-react-app 其实包括两个核心：

* `create-react-app`：主要提供了 command-line 工具，方便用户创建 react 项目
* `react-scripts`：这才是核心。它封装了所有开发 react 项目的配置，使得用户可以零配置直接开始开发 react。用户基本不需要更新 `create-react-app`，因为它总是拉最新的 `react-scripts`，而 `react-scripts` 才是简化用户配置的核心。

问题来了，我在写测试的时候发现，`react-scripts` 默认配置使用的是 jest，而且版本较低（可运行 `npm ls jest` 查看依赖树，结果如下图所示）。而我需要使用的 data-driven-test 依赖包 `jest-each` 是 jest 23.0.0 以后的版本才有的。

![react-scripts-dependency.png](https://upload-images.jianshu.io/upload_images/721960-cc0aae2294d01d0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以我需要在测试的时候不使用默认安装的 jest，


读了一个 [post: 自己写样板，不使用 create-react-app](https://medium.com/@francesco.agnoletto/i-didnt-like-create-react-app-so-i-created-my-own-boilerplate-190a7dd5d74)，受它的启发要自己配置 react 项目，那么就有必要了解 js ecosystem 中的一些工程。

# [webpack](https://www.webpackjs.com/concepts/)

一个开源的前端打包工具，支持用户进行模块化开发（即用户开发很多 module，然后不同的 mudule 之间可通过 import, export 进行相互引用）。原本 js 是不支持的，可参见 [js global namespace](https://hfcherish.github.io/2018/06/21/js-global-namespace/)。

# [ESLint](http://eslint.cn/docs/user-guide/configuring)

是一个 javascript 的语法检查器。类似于在 IDE 中写 java 时的即时编译检查。它是可以配置的，以完成你所需要的检查。

# [ECMAScript](https://zh.wikipedia.org/wiki/ECMAScript)

[ES5, ES6, ES2016, ES.Next: What’s going on with JavaScript versioning?](https://huangxuan.me/2015/09/22/js-version/) 翻译的这个文章讲得很好，我直接引用原文：

>* **ECMAScript**：一个由 ECMA International 进行标准化，TC39 委员会进行监督的语言。通常用于指代标准本身。
>* **JavaScript**：ECMAScript 标准的各种实现的最常用称呼。这个术语并不局限于某个特定版本的 ECMAScript 规范，并且可能被用于任何不同程度的任意版本的 ECMAScript 的实现。
>* **ECMAScript 5 (ES5)**：ECMAScript 的第五版修订，于 2009 年完成标准化。这个规范在所有现代浏览器中都相当完全的实现了。
>* **ECMAScript 6 (ES6) / ECMAScript 2015 (ES2015)**：ECMAScript 的第六版修订，于 2015 年完成标准化。这个标准被部分实现于大部分现代浏览器。可以查阅这张兼容性表来查看不同浏览器和工具的实现情况。
>* **ECMAScript 2016**：预计的第七版 ECMAScript 修订，计划于明年夏季发布。这份规范具体将包含哪些特性还没有最终确定
>* **ECMAScript Proposals**：被考虑加入未来版本 ECMAScript 标准的特性与语法提案，他们需要经历五个阶段：Strawman（稻草人），Proposal（提议），Draft（草案），Candidate（候选）以及 Finished （完成）。

blog 里也讲了 ecmascript 的历史。简答总结一下：

1. 很久以前（1996），网景浏览器把他们写的 javascript 交给 ECMA International（欧洲计算机制造协会）进行标准化
2. ECMA 1，2，3 版本很快发布，而且被各大浏览器厂商支持
3. ECMA 一直没有变化，各大浏览器厂商自行扩展（所以 javascript 其实是 ECMAScript 的实现和扩展）
4. 某一年，ECMA 4 出来，太激进，被废弃了（只有 Adobe 实现了）
5. 2009 年，ECMA 5（es5） 出来。但直到 2012 年才逐渐被公众接受
6. 2015 年，es6 出来。与此同时，相关委员会决定每年定义一次新标准，避免等待整个草案完成。因此 ES6 也被命名为 ECMAScript 2015

一些资源：

* 如果你还不熟悉 ES6，Babel 有一个[很不错的特性概览](https://babeljs.io/docs/learn-es2015/)
* 如果你希望深入 ES6，这里有两本很不错的书： Axel Rauschmayer 的 [Exploring ES6](http://exploringjs.com/)和 Nicholas Zakas 的 [Understanding ECMAScript 6](https://leanpub.com/understandinges6)。Axel 的博客 [2ality](http://www.2ality.com/) 也是很不错的 ES6 资源

# [babel](https://www.babeljs.cn/)

babel 是一个 javascript 编译器。其核心是一个语法转换器，能将使用 es6、flow、jsx 的代码转换为浏览器兼容的 javascript。所以这些项目 build 后其实是生成了浏览器兼容的 javascript。

为啥会有这个，原因就是上边说的，新版本 js 出来了，但是大家还不支持。