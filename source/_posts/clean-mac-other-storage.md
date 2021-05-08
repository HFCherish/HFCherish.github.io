---
title: clean mac other storage
toc: true
date: 2021-02-24 19:43:25
tags:
---


other storage 主要是存的系统的一些缓存、日志等数据。有时会占特别大空间，可以按下列步骤清理

## 1. 暂时关闭 SIP，以能查看和删除系统文件（解决 not permitted 问题）

1. 以 recover mode 重启电脑：启动时，按 command + R 即可
2. 选择 Utilities -> Terminal
3. 在 Terminal 中输入 `csrutil disable` 关闭 SIP
4. 重启电脑

在完成 clean 后，应该重复 1、2，并在 terminal 中输入 `csrutil enable` 来启动 SIP

重启电脑后，可以通过 `csrutil status` 来查看 SIP 服务是否启动（清理完成后应该启动）

## 2. 按 size 查找大文件

```sh
$ cd /
$ sudo du -sh  -- *| sort -hr
```

## 3.  常见的大文件

### ~/Libraray/Caches 和  /Library/Caches

`~/Libraray` 和  `/Library` 下的 `Caches` 和 `logs` 等都是可以安全删除的。可以查看一下大小，把自己不用的 cache 删掉。

当然也可以查看 `Library` 下的所有大文件，确认是否可以删除

### docker

Docker 的 images、volumes 等可能占很大空间，可以查看到 `~/Library/Containers/com.docker.docker` 文件夹的大小

[docker space for mac](https://docs.docker.com/docker-for-mac/space/)

```sh
$ cd ~/Library/Containers
$ sudo du -sh  -- *| sort -hr

# 查看 docker 的系统占用，这是清理后了，占用很小
$ docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              1                   1                   100.8kB             0B (0%)
Containers          1                   0                   0B                  0B
Local Volumes       0                   0                   0B                  0B
Build Cache         0                   0                   0B                  0B

# 清理磁盘，删除关闭的容器、无用的数据卷和网络，以及 dangling 镜像(即无 tag 的镜像)
$ docker system prune
# 清理得更加彻底，可以将没有容器使用 Docker 镜像都删掉
$ docker system prune -a
# 如果需要同时删除未被任何容器引用的数据卷需要显式的指定 --volumns 参数
$ docker system prune --all --force --volumes
# 删除所有 dangling 数据卷(即无用的 volume)
$ docker volume rm $(docker volume ls -qf dangling=true)

# 有时删完后，需要一段时间 reclaim space，可以使用以下命令，手动 trigger relamation
$ docker run --privileged --pid=host docker/desktop-reclaim-space
```

### /Library/Updates

这里可能会有很多文件，都可以删。这里正常应该在执行 App Store 里的 Update 时，才会有文件，但不知道为啥，即使 App Store 没有 Update 也会有。

如果这里有大文件，那么：

1. 先尝试去执行 App Store 里的 Update
2. 如果还有文件，可以删除文件夹里的所有文件（建议不要删那几个 `.plist` 文件），不要删除 `/Library/Updates` 文件夹本身，删除里边的内容

### /private/var/tmp/WiFiDiagnostics*

`/private/var/tmp/` 里的文件删除的时候要小心些。

`WiFiDiagnostics` 是 wifi log，这些文件**可以安全删除**

但它可以用  `command+control+option+shift+w` 或 `command+control+option+shift+>` 触发。如果你正好使用 karabiner 并且 remap 了 `command+control+option+shift`，在使用过程中可能正好和 +w 或 +> 冲突了，那么每次都会启动 logging。所以需要修改配置。

1. karabiner -> Misc -> Open config folder
2. open karabiner.json，在其中加入下面的配置

```json
"rules": [
    {
        "description": "Disabling command+control+option+shift+w. This triggers wifi logging.",
        "manipulators": [
            {
                "from": {
                    "key_code": "w",
                    "modifiers": {
                        "mandatory": [
                            "command",
                            "control",
                            "option",
                            "shift"
                        ]
                    }
                },
                "to": [
                    {
                        "key_code": "escape"
                    }
                ],
                "type": "basic"
            }
        ]
    },
    {
        "description": "Disabling command+control+option+shift+>. This triggers wifi logging also.",
        "manipulators": [
            {
                "from": {
                    "key_code": ">",
                    "modifiers": {
                        "mandatory": [
                            "command",
                            "control",
                            "option",
                            "shift"
                        ]
                    }
                },
                "to": [
                    {
                        "key_code": "escape"
                    }
                ],
                "type": "basic"
            }
        ]
    },
    {
        "description": "Change caps_lock key to command+control+option+shift. (Post escape key when pressed alone)",
        "manipulators": [
            {
                "from": {
                    "key_code": "caps_lock",
                    "modifiers": {
                        "optional": [
                            "any"
                        ]
                    }
                },
                "to": [
                    {
                        "key_code": "left_shift",
                        "modifiers": [
                            "left_command",
                            "left_control",
                            "left_option"
                        ]
                    }
                ],
                "to_if_alone": [
                    {
                        "key_code": "escape"
                    }
                ],
                "type": "basic"
            }
        ]
    }
]
```

> `/private/var/tmp` 文件夹下可能还有很多 `sysdiagnose` 文件，这个应该是可以删，是系统诊断结果。但不太确定，我没删。
>
> Sysdiagnose 可以通过 `command+control+option+shift+.` 来启动一次，所以也要注意