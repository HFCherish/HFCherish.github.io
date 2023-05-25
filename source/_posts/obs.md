---
title: obs
toc: true
tags:
  - cloud
  - storage
date: 2019-03-01 10:50:15
---

[Huawei Obs](https://support.huaweicloud.com/productdesc-obs/zh-cn_topic_0045829060.html) is an object storage service on cloud.

# Concepts

## Object

1. The real complete file or byte stream to save
2. **object name is the unique id** in a bucket
   1. it's used as **part of url path**. The naming restrictions are fit to url path naming restrictions.
3. Access(based on version in fact)
   1. Object ACL:
      1. general control to object: **read object, read/write object ACL, only users in the same account**
   2. Object policy
      1. fine-grained control to object: **fine-grained actions(put,delete...) on object, all users**
4. **multi-versions**
   1. an object can has multiple versions, each of which has an unique id.
   2. Whether there's multi-version, it's a policy set on a bucket.
5. directory:
   1. **directory is just a view**. Essentially, it's an empty object end with "/".
   2. **all objects in a bucket are on the same level**. There's no multi-level directory in fact.
   3. to create the directory view, you need to create an object with name ending with "/" explicility, eg. "sub1/sub2/ . It will create a two-level dir in console. There's no need to create "sub1/" first then "sub1/sub2".
6. object actions:
   1. For writing, there's **only write/restricted-append/delete, no put**
   2. **basically-write-once-read-many**
7. upload modes:
   1. stream
   2. file
   3. multi-part (support breakpoint resume)
   4. append

## Bucket

1. The place to save objects
2. **bucket name is the unique id** for one account(a tenant).
   1. it's used as **part of domain name** on url. The naming restrictions are fit to domain naming restrictions.
3. Access
   1. Bucket ACL: 
      1. general control to bucket and all objects in bucket: **read/put buckets, read/write bucket ACL, only users in the same account**
   2. Bucket policy
      1. fine-grained control to specific objects in bucket: **fine-grained actions on bucket or specific objects in bucket, all users**
4. storage type
   1. standard: 
      1. **quick access & high throughput**. It's used for high access requests and not so big files.
   2. warm:
      1. **low access.**
   3. cold:
      1. **very very low access**

## region

The region of nodes where the storage really happens.

## signature

The signature to identify a user when accessing buckets/objects.

1. ak(access key): represent a user. one user can have multi aks. It's kind of like an user role
2. sk(secret key): one-to-one corresponding with ak. The secret key used for RSA authentication & authorization.