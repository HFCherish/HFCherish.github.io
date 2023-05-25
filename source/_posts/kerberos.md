---
toc: true
tags:
  - hadoop
  - security
stitle: kerberos
date: 2021-05-17 10:14:44
title:
---


[图解Kerberos协议原理](http://www.nosqlnotes.com/technotes/kerberos-protocol/)

[内网渗透之kerberos协议分析](https://xz.aliyun.com/t/8187)

[kerberos 协议](https://zhuanlan.zhihu.com/p/96032550)

[Kerberos Concepts - Principals, Keytabs and Delegation Tokens](https://docs.cloudera.com/documentation/enterprise/5-6-x/topics/cm_sg_principal_keytab.html)

> A user in Kerberos is called a principal, which is made up of three distinct components: the primary, instance, and realm. A Kerberos principal is used in a Kerberos-secured system to represent a unique identity. The first component of the principal is called the primary, or sometimes the user component. The primary component is an arbitrary string and may be the operating system username of the user or the name of a service. The primary component is followed by an optional section called the instance, which is used to create principals that are used by users in special roles or to define the host on which a service runs, for example. An instance, if it exists, is separated from the primary by a slash and then the content is used to disambiguate multiple principals for a single user or service. The final component of the principal is the realm. The realm is similar to a domain in DNS in that it logically defines a related group of objects, although rather than hostnames as in DNS, the Kerberos realm defines a group of principals . Each realm can have its own settings including the location of the KDC on the network and supported encryption algorithms. Large organizations commonly create distinct realms to delegate administration of a realm to a group within the enterprise. Realms, by convention, are written in uppercase characters.

Kerberos assigns tickets to Kerberos principals to enable them to access Kerberos-secured Hadoop services. For the Hadoop daemon principals, the principal names should be of the format service/fully.qualified.domain.name@YOUR-REALM.COM. Here, service in the service/fully.qualified.domain.name@YOUR-REALM.COM principal refers to the username of an existing Unix account that is use    d by Hadoop daemons, such as hdfs or mapred.

> 

Human users who want to access the Hadoop cluster also need to have Kerberos principals of the format, username@YOUR-REALM.COM; in this case, username refers to the username of the user's Unix account, such as joe or jane. Single-component principal names (such as joe@YOUR-REALM.COM) are typical for client user accounts. Hadoop does not support more than two-component principal names.

TGT类似于护照，Ticket则是签证，而访问特定的服务则好比出游某个国家。与护照一样，TGT可标识你的身份并允许你获得多个Ticket（签证），每个Ticket对应一个特定的服务，TGT和Ticket同样具有有效期，过期后就需要重新认证。

# impersonate

[secure impersonate](http://hadoop.apache.org/docs/r1.2.1/Secure_Impersonation.html

[proxy user](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/Superusers.html#Configurations)

# kerberos 常用命令

```shell
# 进入kerberos控制台
kadmin.local

#添加用户，增加实例 add_principal, addprinc, ank
addprinc -randkey xxxx@xxxxT.COM

# 为实例生成秘钥
xst -k xxxx.keytab xxxx@xxxxT.COM
或
kadmin.local -q "xst -k xxxx.keytab xxxx@xxxxT.COM"

# 合并hadoop.keytab和HTTP.keytab为hdfs.keytab
ktutil
rkt hadoop.keytab
rkt HTTP.keytab
wkt hdfs.keytab

# 查看keytab文件
klist -ket /etc/krb5.keytab

# 删除用户 delete_principal, delprinc
delprinc xxxx@xxxxT.COM

# 修改用户密码 change_password, cpw
cpw xxxx@xxxxT.COM

# 查看所有用户 list_principals, listprincs, get_principals, getprincs
listprincs

# 修改票据属性 modify_principal, modprinc
modprinc -maxrenewlife 1week  可在一周内renew

# 查看票据信息 get_principal, getprinc
getprinc xxxx@xxxxT.COM

# 导出keytab文件 ktadd, xst
 xst -e aes128-cts-hmac-sha1-96:normal -k /home/dengsc/dengsc.keytab dengsc@XXXX.COM
        -e 执定加密方式
        -k 指定keytab文件名
        注:导出keytab文件时会重新生成密码.
            kadmin.local模式下可添加参数‘-norandkey’,导出keytab文件时不重置密码.
            egg: xst -norandkey -k /home/dengsc/dengsc.keytab
```

# SSL 证书

## Keystore 常用命令

```sh
# 生成jks
keytool -genkeypair -alias prestoservernew -keyalg RSA -keystore presto_keystore_new.jks

# 导出cer证书
keytool -keystore /javahome/jre/lib/security/presto_keystore_new.jks -export -alias prestoservernew -file /tmp/prestoservernew.cer

# 导入证书到truststore
keytool -import -keystore /javahome/jre/lib/security/cacerts -trustcacerts -alias prestoservernew -file /tmp/prestoservernew.cer

# 删除信任证书
keytool -delete -alias prestoservernew -keystore  /javahome/jre/lib/security/cacerts


#检查导入成功：
keytool  \
-keystore /javahome/jre/lib/security/cacerts  \
-storepass changeit \
-list

# 删除
keytool -delete -alias ldapservern2 -keystore  /javahome/jre/lib/security/cacerts
#changeit

#查看证书
ubuntu@prestoserver:~/data/presto-server-0.215/etc$ keytool -printcert -v -file   /javahome/jre/lib/security/dc.cer
Owner: CN=al.al.com
Issuer: CN=al-AL-CA-1, DC=al, DC=com
```

## .cer vs .crt

[Do I need to convert .CER to .CRT for Apache SSL certificates? If so, how?](https://stackoverflow.com/questions/642284/do-i-need-to-convert-cer-to-crt-for-apache-ssl-certificates-if-so-how)

**CER** is an X.509 certificate in binary form, **DER** encoded.
**CRT** is a binary X.509 certificate, encapsulated in text (**base-64**) encoding.

可以互相转：

```shell
# When type DER returns an error loading certificate (asn1 encoding routines), try the PEM and it shall work.
openssl x509 -inform DER -in certificate.cer -out certificate.crt
openssl x509 -inform PEM -in certificate.cer -out certificate.crt
# pem
openssl x509 -inform DER -in certificate.cer -out certificate.pem
```

> File extensions for cryptographic certificates aren't really as standardized as you'd expect. Windows by default treats double-clicking a `.crt` file as a request to import the certificate into the Windows Root Certificate store, but treats a `.cer` file as a request just to view the certificate. So, they're different in the sense that Windows has some inherent different meaning for what happens when you double click each type of file.

But the way that Windows handles them when you double-click them is about the only difference between the two. Both extensions just represent that it contains a public certificate. You can rename a certificate file to use one extension in place of the other in any system or configuration file that I've seen. And on non-Windows platforms (and even on Windows), people aren't particularly careful about which extension they use, and treat them both interchangeably, as there's no difference between them as long as the contents of the file are correct.

> 

Making things more confusing is that there are two standard ways of storing certificate data in a file: One is a "binary" X.509 encoding, and the other is a "text" base64 encoding that usually starts with "`-----BEGIN CERTIFICATE-----`". These encode the same data but in different ways. Most systems accept both formats, but, if you need to, you can convert one to the other via openssl or other tools. The encoding within a certificate file is really independent of which extension somebody gave the file.

### 默认的 certificate 目录

下边这条命令可以打出目录，这里都是 symbolic link，继续追踪，可以看到文件最终是存放在  `/etc/pki/ca-trust/extracted`，但是这个文件夹的内容是自动生成的（可以查看该文件夹下的 README）。当执行 `update-ca-trust` 命令时，会根据 `/usr/share/pki/ca-trust-source/` and `/etc/pki/ca-trust/source/` 自动生成 `extracted` 目录。

```shell
$ python3 -c "import ssl; print(ssl.get_default_verify_paths())"
DefaultVerifyPaths(cafile='/etc/pki/tls/cert.pem', capath='/etc/pki/tls/certs', openssl_cafile_env='SSL_CERT_FILE', openssl_cafile='/etc/pki/tls/cert.pem', openssl_capath_env='SSL_CERT_DIR', openssl_capath='/etc/pki/tls/certs')
```

如果需要修改证书，可以使用 [update-ca-trust](https://www.linux.org/docs/man8/update-ca-trust.html) 命令。
