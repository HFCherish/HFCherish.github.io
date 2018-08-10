---
title: oauth
toc: true
date: 2018-07-27 15:24:55
tags:
	- security
	- oauth
---

# traditional authentication

传统认证使用  session:

1. client 发送 username、password 给 server
2. server 查数据库，检查信息，是否正确。正确就把用户登录信息(即用户状态)写到 session 里（即服务器内存中），并将 sessionId 返回给 client。
3. client 在请求 api 时，在 cookie 中传递 sessionId。server 端根据 sessionId 获取用户登录信息，如果已认证，返回正常响应；反之，401

![auth image](https://camo.githubusercontent.com/f6ea1099ada7ec855919d6e483d0d903f1cc96ca/68747470733a2f2f636d732d6173736574732e74757473706c75732e636f6d2f75706c6f6164732f75736572732f3438372f706f7374732f32323534332f696d6167652f747261646974696f6e616c2d61757468656e7469636174696f6e2d73797374656d2d706e672e706e67)

这种方式有个缺陷：如果做分布式服务部署，那么需要每个服务器都要同步相同的登录信息，这不是一个好的方式。所以一般 rest 微服务都要求的是 stateless，即 server 端不保存任何用户信息，请求中包含所有需要的信息。

# oauth

[oauth](https://zh.wikipedia.org/wiki/%E5%BC%80%E6%94%BE%E6%8E%88%E6%9D%83) 是一个开放标准，允许用户让第三方应用访问该用户在某一网站上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和[密码](https://zh.wikipedia.org/wiki/%E5%AF%86%E7%A0%81)提供给第三方应用。

OAuth允许用户提供一个 [令牌](https://zh.wikipedia.org/w/index.php?title=%E4%BB%A4%E7%89%8C&action=edit&redlink=1)，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的网站（例如，视频编辑网站)在特定的时段（例如，接下来的2小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。这样，OAuth让用户可以授权第三方网站访问他们存储在另外服务提供者的某些特定信息，而非所有内容。

 其令牌可以是 JWT 或其他形式。

![oauth](https://docs.oracle.com/cd/E74890_01/books/RestAPI/images/OAuth2leg_V.gif)

ref: [oauth 2.0 的用途](https://my.oschina.net/eyes4/blog/639970)

# Auth0

[autho0](https://auth0.com/) 实现了[很多开放标准](https://auth0.com/docs/protocols)，包括 oauth。([学习视频](https://www.youtube.com/watch?v=QsMK3d3LxYQ))

1. 要使用 Auth0，首先需要创建一个 App（被称作 client），其中定义了 clientId、domain name、callbackUrl、secret 等。
2. 前端交互：
   1. 当访问某个页面时，查看 localstorage，看用户是否登录；
   2. 如果未登录，利用 Auth0 sdk 或 api 登录认证（提供前边 App 中的 clientId、secret 等信息），认证通过将认证信息（token 等）存入 localstorage，并跳转到 callback url
   3. 如果登录，直接访问
3. 后端交互：
   1. 前端携带 token 访问 API
   2. server 利用 Auth0 sdk 或 api 验证 token 的有效性；认证通过返回资源，否则 401