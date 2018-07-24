---
title: hexo+next 设置
toc: true
date: 2018-06-14 19:19:44
tags:
	- tool
---

[使用 hexo + github 部署博客](https://hfcherish.github.io/2018/05/23/build-blog-using-hexo/) 介绍了怎么部署自己的博客，然后就开始无休止的调整主题。

我选定的主题是 [next](https://github.com/theme-next/hexo-theme-next)：有目录，也有集成搜索的文档，这是一个 [example](http://www.itfanr.cc/about/)，参照 [第三方集成](https://theme-next.iissnan.com/third-party-services.html) 集成搜索等功能. next 优化配置可参考 [这篇文章](http://www.vitah.net/posts/20f300cc/)

# 配置文件

我是采用两个配置文件的写法，即在 `source/_data/next.yml` 中写 next 相关的配置。

# code highlight 配置

按照[主题设定教程](https://theme-next.iissnan.com/getting-started.html#theme-settings)，我设置的是 `scheme: Mist`。默认的代码 highlight 是用 [tomorrow theme](https://github.com/chriskempson/tomorrow-theme)，按照 [代码高亮设置教程](https://theme-next.iissnan.com/theme-settings.html#syntax-highlight-scheme)，可以有五种选项。但是很多 code grammar 高亮显示无效，比如 jsx。所以想换一个 highlight 主题。

找了个 [code highlight theme 配置教程](https://vxhly.github.io/2017/10/hexo-next-advanced-settings/) ，开始动手。

