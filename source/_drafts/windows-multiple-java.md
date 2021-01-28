---
title: 'windows multiple java'
toc: true
date: 2020-12-18 18:21:15
tags:
	- windows
	- java
---

errors:

```
Registry key 'Software\JavaSoft\Java Runtime Environment\CurrentVersion' has value '1.8', but '1.6' is required.
```

或者 java 和 javac 的版本不一样. 修改：

1. 修改 registry key 的 `HKEY_LOCAL_MACHINE\SOFTWARE\JavaSoft\Java Runtime Environment\`  的 `currentVersion` 和 `Java6FamilyVersion` 为一致

   > key 有可能在 `HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\JavaSoft\Java Runtime Environment` 在 64 位 win10 系统中

2. 删除环境变量中 `C:\Program Files (x86)\Common Files\Oracle\Java\javapath`

3. 删除上述 javapath 下的三个 exe（java.exe, javaw.exe, javaws.exe）

4. 删除 C:\Windows\SysWOW64 (或 System32) 下的上述三个 exe

![image-20201218183130885](/Users/zhenzheng/Library/Application Support/typora-user-images/image-20201218183130885.png)

![image-20201218183335843](/Users/zhenzheng/Library/Application Support/typora-user-images/image-20201218183335843.png)

[控制 ie 的 java 配置](https://bbs.csdn.net/topics/300069171)

[java control panel](https://zh.wikihow.com/%E5%90%AF%E7%94%A8Java)



CLASSPATH

.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;



JAVA_HOME

C:\Program Files (x86)\Java\jdk1.6.0_45



%JAVA_HOME%\bin

%JAVA_HOME%\jre\bin