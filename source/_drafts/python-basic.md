---
title: python basic
toc: true
date: 2019-03-26 12:43:42
tags:
	- python
---

# pip

[pip](https://pypi.org/project/pip/) is the package installer for python.

```sh
$ pip install package-name
```

On windows, to use `pip` after python installation, you need to config both the python & pip path.

```sh
$ export PATH=$PATH:c/users/xiaoming/AppData/Programs/Python/Python37:c/users/xiaoming/AppData/Programs/Python/Python37/Scripts
```

# setup.py

[setuptools](https://setuptools.readthedocs.io/en/latest/) is the packaging tool for python (like maven/gradle for java?)

When coding in local, you can use `pip install` to install dependency. However, when deploy a python project, you need to package all dependency, modules into one project. You need `setup.py`.

When organize python in modules, you need to know and import the function in other modules. In compilation stage, you need to feel the other modules&packages. `setup.py` does the magic. (This is my guess)

# if \__name__ == '\_\_main__'

[如何简单地理解Python中的if \__name__ == '\_\_main__'](https://blog.csdn.net/yjk13703623757/article/details/77918633)

If you are `test.py`, then other module knows you as `__name__=='test'`, and you know yourself as `__name=='__main__'`.

The code block in `if __name__ == '__main__'` will only be executed when the file is executed directly. If other module import `test.py` as module, the code block won't be executed.

It's kind of like main function in java/c, but with big difference. Python is **a script language** — interpretive execution, which can be run without any so-called main entry. You don't need to make `test.py` runnable by providing `main`, but you can provide `if __name__ == '__main__'` if you want this block only executed when called directly rather than as module.

