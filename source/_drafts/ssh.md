---
title: ssh
toc: true
date: 2019-01-07 11:08:55
tags:
	- ssh
---

[ssh keys](https://wiki.archlinux.org/index.php/SSH_keys)

# ssh-keygen

可以用于生成公钥（public key）和密钥，用于认证。

ssh-keygen 支持多种算法生成公钥（详情见 `man ssh-keygen`），默认是 RSA。RSA key length 最小是768 bits，默认是 2048 bits，长度无限制。DSA（digital signature algorithm）最大长度是 1024 (see [ssh-keygen tutorial](https://www.guyrutenberg.com/2007/10/05/ssh-keygen-tutorial-generating-rsa-and-dsa-keys/))。两种算法的安全性似乎差不多，DSA 生成密钥的速度快一些，validation 慢一些，RSA 则相反。大部分情况下，RSA 能满足要求。

公钥和密钥保存在 `~/.ssh/` 下，文件名称相同，公钥文件有一个后缀 `.pub`。将公钥给第三方程序，用于认证。

# sshd

From `man sshd`:

> sshd (OpenSSH Daemon) is the daemon program for ssh.
>
> sshd listens for connections from clients. It's normally started at boot from /etc/rc. It forks a new daemon for each incoming connection.

# ssh localhost without password

It seems that only when create key in file id_rsa or id_dsa, the command will work. Other file name won't take effect.

```sh
$ ssh-keygen -t rsa
$ cat ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```



