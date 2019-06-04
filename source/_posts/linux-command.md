---
title: linux commands
toc: true
tags:
  - linux
date: 2019-01-23 15:32:13
---


# chmod, chown

[understanding linux file permissions](https://www.linux.com/learn/understanding-linux-file-permissions)

File permissions are defined by **permission group** and **permission type**

1. permission group
   * owner(u)
   * group(g)
   * all other users(a)
2. permission type
   * read (r - 4)
   * write(w - 2)
   * execute(x - 1)

## permission presentation

The permission in the command line is displayed as ***_rwxrwxrwx 1 owner:group***

* the first character (underscore **_**  here) is the **special permission flag** that can vary.
* the following three groups of ***rwx*** represent **permission of owner, group and all other users** respectively. If the owner and all users has no read permission, it is ***__wxrwx_wx***
* follwing that grouping since the integer displays **the number of hardlinks to the file**
* the last piece is the owner and group assignment.

### special permission flag

The special permission flag can be:

* **_**: no special permissions
* ***d***: directory
* ***l***: the file or dir is a symbolic link
* ***s***: This indicated the setuid/setgid permissions. This is not set displayed in the special permission part of the permissions display, but is represented as a **s** in the read portion of the owner or group permissions.
* ***t***: This indicates the sticky bit permissions. This is not set displayed in the special permission part of the permissions display, but is represented as a **t** in the executable portion of the all users permissions

## permission modification

```sh
# grant read and write permissions to the user and group
$ chmod ug+rw file1

# remove read and write permissions to the user and group
$ chmod ug-rw file1

# set permission using binary references (owner: rwx = 4+2+1, group: rx = 4+1, all users: rx = 4+1)
$ chmod 755 file1

# change the file permission recursively in the file/dir instead of just the files themselves
$ chmod -R 755 dir1
```

## change owner:group assignments

```sh
# change the owner of file1 to user1 and group to family
$ chown user1:family file1
```

# find

```sh
# find all xml files from the current dir
$ find ./* -name '*.xml'
```

