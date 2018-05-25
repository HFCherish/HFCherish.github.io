---
title: 使用 hexo + github 部署博客
---

# 安装部署 [hexo](https://hexo.io/zh-cn/docs/index.html)

参考这个[知乎文章安装](https://zhuanlan.zhihu.com/p/26625249)

## 1. 准备 node、git
## 2. 安装 hexo-cli

```sh
$ npm i -g hexo-cli
```

## 3. 创建一个网站
```
$ hexo init xxx.github.io
```

## 4. [部署服务器](https://hexo.io/zh-cn/docs/deployment.html)

这里选择部署到 git 上。

1. 首先安装 git deployer
2. 然后修改配置文件，选择部署方式为 git 并配置 repo。
3. hexo 部署
4. 访问 `xxx.github.io` 即可

```
$ npm install hexo-deployer-git --save
$ vi _config.yml

deploy:
  type: git
  repo: <repository url>
  branch: [branch]
  message: [message]

$ npm d
```

注意：

>1. 可以选择同时部署到多个服务器。多写几个 'deploy' 配置即可
>2. git 用户名、网站用户名（`xxx.github.io` 中的 `xxx`）必须相同。因为它相当于使用 github 服务器

# 设置 theme

在 [官方 themes](https://hexo.io/themes/index.html) 里挑。我比较喜欢以下几款：

1. 带目录结构的：
	2. [cactus](https://github.com/probberechts/hexo-theme-cactus/blob/master/README.md)：英文的，有几种颜色可以选，带目录，可以配置搜索，简洁，这是[white 版本的](https://probberechts.github.io/hexo-theme-cactus/cactus-white/public/archives/)
	3. [aircloud](https://github.com/aircloud/hexo-theme-aircloud)：英文中文都 ok，有目录，还可以搜索
	4. [next](https://github.com/theme-next/hexo-theme-next)：有目录，也有集成搜索的文档，这是一个 [example](http://www.itfanr.cc/about/)，参照 [第三方集成](https://theme-next.iissnan.com/third-party-services.html) 集成搜索等功能
	5. [yilia](https://github.com/litten/hexo-theme-yilia)：有目录，有搜索，[owner 博客](http://litten.me/2017/12/29/diary-2017-1222-1229/)
3. 没有目录，没有 tag
	3. [apollo](https://github.com/pinggod/hexo-theme-apollo)：[blog](http://pinggod.com/archives/) 是中文的，比较简单，颜色也好看

配置很简单，以 [yilia](https://github.com/litten/hexo-theme-yilia) 为例：

```sh
# 1. 找到 theme git，download 到 `themes` 文件夹下
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia

# 2. 修改 hexo 根目录下的配置文件，指定使用该主题
$ vi _config.yml
theme: yilia

# 3. customize 主题配置. 可以修改 themes 中的 config，也可以直接在 hexo config 中修改
## 修改 themes 中的 _config.yml
$ vi themes/yilia/_config.yml
...
...
...
## 修改 hexo config
$ vi _config.yml
theme_config:
	...
	...
	...

```

# 发表博客

## 1. [创建博客](https://hexo.io/zh-cn/docs/commands.html#new)
利用命令创建一个博客，存放在 `source/_posts/` 下。然后可以编辑这个文件。刷新页面就可以看到博客有更新了

```
$ hexo new titlename
```

也可以直接把 md 文件 copy 到 `source/_posts/` 下，可以添加 [`frong-matter`](https://hexo.io/zh-cn/docs/front-matter.html) 指定 category、tag 等。

## 2. [生成静态文件](https://hexo.io/zh-cn/docs/generating.html)
`hexo g` 会根据 md 生成 html、css 等静态文件。文章写完后，利用这个命令生成静态文件，然后再 `hexo d` 部署即可。

也可以使用下述命令（两个命令等价），指同时 `generate` + `deploy`

```
$ hexo g -d
$ hexo d -g
```

### hexo clean

有时可能需要使用 [`hexo clean`](https://hexo.io/zh-cn/docs/commands.html#clean) 清除缓存文件 `db.json` 和已生成的静态文件 `public`

```
$ hexo clean
```

### [启动本地服务器（调试用）](https://hexo.io/zh-cn/docs/commands.html#server)

博客编辑完成后，可以先本地启动 server，看一下效果

```
$ hexo s
```

`-s` 参数指定仅仅启动静态模式，即创建博客后，必须要 `hexo g` 去生成 `index.html` 等（相当于发布），网站才会真正的更新。否则网站不会更新。这一般用于 `production mode`

一般编辑完文章后，可以先本地启动服务器，调试一下样式可不可以，然后再部署到服务器上
