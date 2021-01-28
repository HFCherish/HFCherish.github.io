---
title: atlas
toc: true
tags:
  - metadata
  - big data
date: 2020-11-13 10:04:33
---


# Architecture

![img](http://atlas.apache.org/public/images/twiki/architecture.png)

# Install

[install steps](https://atlas.apache.org/2.0.0/InstallationSteps.html)

Access Apache Atlas UI using a browser: [http://localhost:21000](http://localhost:21000/) 

You can also access the rest api `http://localhost:21000/api/atlas/v2`

默认的用户名密码是 (admin, admin)

# Atlas Features

## 定义元模型，规范元数据

atlas 可以维护（增删改查） metadata types，支持

* 创建多种类型的 metadata types
  * businessmetadatadef：业务元数据的元模型
  * classificationdef：标签数据的元模型
  * entitydef：一般元数据的元模型
  * enumdef
  * relationshipdef：关系元数据的元模型
  * structdef
* 元模型支持定义属性约束、索引、唯一性等
* 按 id/typename/query 来检索

> [相关 API 定义](http://atlas.apache.org/api/v2/resource_TypesREST.html#resource_TypesREST_createAtlasTypeDefs_POST)
>
> [typedef request schema object](http://atlas.apache.org/api/v2/json_AtlasTypesDef.html)
>
> ```
> # DELETE/GET/POST/PUT
> /v2/types/typedef
> ```

### 约束

* typename 全局唯一

## 可以维护元数据

### import metadata

atlas 提供以下途径将元数据引入系统：

1. REST API：atlas 提供 api 可以 bulk saveOrUpdate 某个 type 的元数据
2. 文件：atlas 可以上传文件，并 saveOrUpdate 文件中所定义的元模型、元数据等
3. atlas hook：atlas 通过监听 kafka topic `ATLAS_HOOK` ，来实时引入数据源中的元数据。目前已提供 Apache Hive/Apache HBase/Apache Storm/Apache Sqoop 的 hook
   1. hive hook
      1. 可以 import hive databases & tables 元数据
      2. 可以监听以下类型的 hive 操作，capture 其中的元数据：
         1. create database
         2. create table/view, create table as select
         3. load, import, export
         4. DMLs (insert)
         5. alter database
         6. alter table (skewed table information, stored as, protection is not supported)
         7. alter view
   2. sqoop
      1. 目前仅支持监听 sqoop 的 hiveimport operation 完成后，capture 其中的元数据。

## 业务元数据

atlas 可以:

1. 定义业务元数据元模型规范业务元数据
2. 在技术元数据上，添加业务元数据，来实现关联
3. 支持按业务元数据检索

## 血缘

atlas 可以查询元数据血缘关系。应该是基于关系图实现。

目前 hive 表可以支撑到 column 级别的血缘分析。

## 可以维护标签

atlas 中的 classification 即标签，可以打在元数据、术语等地方。

可以基于 classification 检索元数据。

可以做 classification 的传播：

* 基于继承关系链的传播
* 基于血缘关系链的传播

### Classification vs label

Atlas 中的 classification 和 label 都是标签的概念，label 是轻量级、谁都可以加的简单标签，classification 则有更多的支持。

[atlas classification vs label](https://docs.cloudera.com/runtime/7.2.1/atlas-working-with-classifications/topics/atlas-working-with-classifications.html)

## 可以维护企业术语表

atlas 可以维护企业的术语，并将术语与元数据关联，支持按术语检索元数据，通过以下三个概念来实现细粒度的术语管理：

1. glossary：术语表，最高级别，所有的术语都必须属于某个术语表
2. category：术语分类，必须挂在某个 glossary。可以拥有 childCategories
3. term：术语。必须挂在某个 glossary。可以挂在某个 category

## Notifications

atlas 通过 kafka 实现 hook，引入元数据；也通过 kafka 广播元数据修改，供消费者使用。

* notifications to atlas：`ATLAS_HOOK`. 目前已提供 Apache Hive/Apache HBase/Apache Storm/Apache Sqoop 的 hook，来监听这些数据源的元数据

* notifications from atlas: `ATLAS_ENTITIES`

  ```
  # 监听并发布以下事件的通知
  ENTITY_CREATE:         sent when an entity instance is created
  ENTITY_UPDATE:         sent when an entity instance is updated
  ENTITY_DELETE:         sent when an entity instance is deleted
  CLASSIFICATION_ADD:    sent when classifications are added to an entity instance
  CLASSIFICATION_UPDATE: sent when classifications of an entity instance are updated
  CLASSIFICATION_DELETE: sent when classifications are removed from an entity instance
  
  # notification data
  AtlasEntity  entity;
  OperationType operationType;
  List<AtlasClassification>  classifications
  ```

## 检索

atlas 所有的元数据存储在图数据库 JanusGraph，而索引数据则存储在 index store（solr / elasticsearch）来做全文检索

atlas 支持以下检索方式：

1. 唯一定位元数据：通过 id
2. basic 检索：基于 type、attributes、classifciation、terms 等 query parameter 做全文检索
3. advance 检索：可以使用 dsl 语言做全文检索

## 高可用

1. 基础设施高可用。
   1. atlas 使用 JanusGraph 存储元数据，并将 HBase（默认，可采用其他数据库）作为 backing store。HBase 本身的高可用特性支撑了 metadata store 的高可用
   2. atlas 使用 solr / elasticsearch 存储元数据索引。这些组件同样支持高可用
2. Web Service 高可用。
   1. 目前 atlas 的 web service 同一时间只能有一个 active instance 响应，以实现元数据维护、缓存等的一致性问题。高可用模式即有多个备用（passive）instances，当 active instance down 后，可以自动切换某个 passive instance，作为新的 active instance。

## 访问控制

atlas 支持非常细粒度的访问控制：

* 元模型：基于某个元模型或某类元模型的访问控制。典型 example：

  > - Admin users can create/update/delete types of all categories
  > - Data stewards can create/update/delete classification types
  > - Healthcare data stewards can create/update/delete types having names start with “hc”

* 元数据：基于元模型、标签、元数据 id 的元数据访问控制。典型 example：

  > - Admin users can perform all entity operations on entities of all types
  > - Data stewards can perform all entity operations, except delete, on entities of all types
  > - Data quality admins can add/update/remove DATA_QUALITY classification
  > - Users in specific groups can read/update entities with PII classification or its sub-classification
  > - Finance users can read/update entities whose ID start with ‘finance’

* admin 操作：可以控制 user/group 来 import/export entities

## UI

有个 ui 可以管理元数据、标签、术语

# 限制

1. web service 只有一个 active instance
2. Typename 全局唯一
3. ui 挺慢的

# 基于 atlas 我们可以做什么

1. 数据集发现
   1. 数据字典浏览和检索
2. 数据集导入和导出
   1. 导出数据集，供系统之间交互使用
   2. 导入数据集（数据标准、指标口径等）
3. 标签管理：
   1. 维护用户画像标签 (增删改查)
   2. 基于标签检索数据集
4. 数据标准维护和浏览：（可以通过术语做？）
   1. 维护数据标准（增删改查）
   2. 可以为数据标准加标签
5. 指标和口径维护和浏览
   1. 维护指标口径（增删改查）
   2. 可以为指标口径加标签
6. 数据集 staticstics
   1. 数据接入和使用情况统计
7. 辅助数据分析师生成分析：
   1. 通过关联技术元数据和业务元数据，当数据分析师按业务语言定义分析后，可以快速查找相关的表、字段等