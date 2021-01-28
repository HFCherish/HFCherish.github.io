---
title: linux 包管理工具
toc: true
date: 2020-12-01 17:35:03
tags:
	- linux
---

# yum

下载 .repo 文件

```sh
# 查看现在已有的列表
$ yum list | grep 'xxx 应用'
```

# rpm

下载 rpm 文件 [installation location](https://centos.pkgs.org/7/epel-x86_64)

```sh
$ rpm -Uvh xxx.rpm

# 如果安装的时候有依赖问题，可以加上 --nodeps --force 参数来 ignore 这些问题
$ rpm -ivh xxx.rpm --nodeps --force
```

# pip

下载 .whl 文件 [install location](https://pypi.org/)

```sh
$ pip install pytest-runner --no-index -f ./xxx.whl
```

