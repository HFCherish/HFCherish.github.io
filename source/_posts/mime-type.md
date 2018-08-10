---
title: mime type
toc: true
date: 2018-07-26 17:52:01
tags:
	- rest
---

[MIME 类型](MIME 类型) 是用一种标准化的方式来表示文档的性质和格式。浏览器一般通过 MIME 类型（而不是文档扩展名）来确定如何处理文档。因此服务器传输数据时，必须设置正确的 MIME 类型。

# 通用结构

```
type/subtype
```

1. 不允许空格
2. 大小写不敏感，一般都是小写

# 独立类型

type 可以是独立类型，表示文件的分类，可以是如下值：

| 类型          | 描述                                                         | 典型示例                                                     |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `text`        | 表明文件是普通文本，理论上是可读的语言                       | `text/plain`, `text/html`, `text/css, text/javascript`       |
| `image`       | 表明是某种图像。不包括视频，但是动态图（比如动态gif）也使用image类型 | `image/gif`, `image/png`, `image/jpeg`, `image/bmp`, `image/webp` |
| `audio`       | 表明是某种音频文件                                           | `audio/midi`, `audio/mpeg, audio/webm, audio/ogg, audio/wav` |
| `video`       | 表明是某种视频文件                                           | `video/webm`, `video/ogg`                                    |
| `application` | 表明是某种二进制数据                                         | `application/octet-stream`, `application/pkcs12`, `application/vnd.mspowerpoint`, `application/xhtml+xml`, `application/xml`,  `application/pdf,``application/json` |

一般是文本，但是具体类型不确定时，就用 `test/plain`；是二进制数据，而类型不确定时，用 `application/octet-stream`

## application/octet-stream

这是应用程序文件的默认值。意思是 *未知的应用程序文件 ，*浏览器一般不会自动执行或询问执行。浏览器会将它作为附件来处理，附件类型等信息通过HTTP头[`Content-Disposition`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition) 设置。

# Multipart 类型

```
multipart/form-data
multipart/byteranges
```

 顾名思义，这是复合文件的一种表现形式，即传递过来的数据有多重类型。典型的如果表单数据可能有 string、文件、视频、音频等。

它由边界线（一个由`'--'`开始的字符串）划分出的不同部分组成。每一部分有自己的实体，以及自己的 HTTP 请求头，[`Content-Disposition`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition)和 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 用于文件上传领域，最常用的 ([`Content-Length`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Length) 因为边界线作为分隔符而被忽略）。

例如，如下表单:

```html
<form action="http://localhost:8000/" method="post" enctype="multipart/form-data">
  <input type="text" name="myTextField">
  <input type="checkbox" name="myCheckBox">Check</input>
  <input type="file" name="myFile">
  <button>Send the file</button>
</form>
```

请求是：

```
POST / HTTP/1.1
Host: localhost:8000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------8721656041911415653955004498
Content-Length: 465

-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myTextField"

Test
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myCheckBox"

on
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myFile"; filename="test.txt"
Content-Type: text/plain

Simple file.
-----------------------------8721656041911415653955004498--
```

