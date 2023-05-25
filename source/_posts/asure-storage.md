---
title: azure storage
toc: true
date: 2018-07-27 11:09:53
tags:
    - cloud
    - azure
---

asure storage 提供四种存储支持（[asure storage overview (youtube)](https://www.youtube.com/watch?v=y6bIUHtdp6Y)）：

* [blob](https://azure.microsoft.com/zh-cn/services/storage/blobs/) (binary large object)：二进制数据存储。有两种：page blog（只能新增/删除/向已有数据附加数据，不能修改数据）；block blog（可以更新）
* [table](https://azure.microsoft.com/zh-cn/services/storage/tables/)：存储表格（nosql）
* [queue](https://azure.microsoft.com/zh-cn/services/storage/queues/)：有库，有 rest api
* [files](https://azure.microsoft.com/en-us/services/storage/files/)：好像主要用在文件共享的时候，就是类似于 windows server 上的文件共享（[smb(server message block)](https://zh.wikipedia.org/wiki/%E4%BC%BA%E6%9C%8D%E5%99%A8%E8%A8%8A%E6%81%AF%E5%8D%80%E5%A1%8A)），实现和使用方式都和 windows server 的文件共享一样。所以需要支持例如按 /servername/filename 等方式来 share 和 使用文件。它是构建于 blob 之上的。

# 几个概念

## Storage Accout

storage 的所有存储都必须在一个 storage account 内发生。这有点类似于一个 database。

安全也是在这里实现：

1. ***key***：创建 account 时，就会生成俩 key，primary key 就是你用来登录访问数据的 key。不过这种方式对于有 client 时不太方便，因为可能不能 share key
2. ***saas token***：就是可以登录认证获得 token，然后拿 token 访问数据。

## 数据容器 + 数据

在每个 storage account 里，你可以创建上述四种类型的存储容器，去存放数据。

* Blob： 在 storage account 里可以创建 ***blob container***，在 ***blog container*** 中存放 ***blob***
* table：在 storage account 里创建 ***table***，在 ***table*** 中存放 ***data entity***
* queue：在 storage account 里创建 ***queue***，在 ***queue*** 中存放 ***message***
* file：在 storage account 里创建 ***asure files*** (类似于 [smb(server message block)](https://zh.wikipedia.org/wiki/%E4%BC%BA%E6%9C%8D%E5%99%A8%E8%A8%8A%E6%81%AF%E5%8D%80%E5%A1%8A))??? 感觉应该也是先创建一个 files container，再创建一个个的 asure file（即 smb）