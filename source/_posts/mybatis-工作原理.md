---
title: mybatis 工作原理
toc: true
date: 2018-08-10 09:58:24
tags:
	- orm
---

# 几个核心类

参见:

* [java api](http://www.mybatis.org/mybatis-3/zh/java-api.html)
* [入门 - 介绍核心使用组件和最佳实践](http://www.mybatis.org/mybatis-3/zh/getting-started.html)

## SqlSessionFactory

mybatis 应用以一个 sqlSessionFactory 实例为核心，即一个应用中有一个**单例** `SqlSessionFactory`，所以数据库 session 都从这里获得。

`SqlSessionFactory` 可以通过 `SqlSessionFactoryBuilder` 获得，builder 负责从 xml 配置或 java configuration 类获得。xml (或相应的 java configuration 类) 配置了 datasource（数据库连接信息）、mappers 等信息

## SqlSessionFactoryBuilder

它主要就是用来获取 `SqlSessionFactory`，可以从 xml 或 Java Configuration 类加载配置并构建。提供如下几种方式来获取（参见[java api](http://www.mybatis.org/mybatis-3/zh/java-api.html)）：

```java
// 从 xml 获取，其中配置了 environment，datasource，mappers  
SqlSessionFactory build(InputStream inputStream);

// 从 xml 获取，但当 xml 配置了多个 env 中的 datasource 等时，通过 env 指定加载的环境
SqlSessionFactory build(InputStream inputStream, String environment);

// 从 xml 获取，但可以指定其中使用到的 properties(详见下文的解释)
SqlSessionFactory build(InputStream inputStream, Properties properties);
    
// 从 xml 获取，并指定 env 和使用的 props
SqlSessionFactory build(InputStream inputStream, String env, Properties props);

// 从 Java Configuration 获取
SqlSessionFactory build(Configuration config)
```

`SqlSessionFactoryBuilder` 只是为了创建 `SqlSessionFactory`，创建完成就可以丢弃 builder 了。所以一般它的生命周期是方法级，是其中的一个局部变量

### 使用 xml

**先配置一个 `config.xml`：**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

上边的配置中，`environments` 配置的是各个环境下的数据库配置。每个环境下，都可以配置 TransactionManager、datasource 等，连接数据库、包括操作数据库的 driver 等信息都是在这里配置的)。

`${driver}` 这种写法是用的 property。property 可以直接在这个 xml 中配置（使用 `<properties>` 标签），也是 java 中的 `System.getProperties()` 中的 property，还可以是在 `SqlSessionBuilder` 中传递的 `props` 参数。

**构建 `SqlSessionFactory`：**

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

### 使用 java Configuration 类

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```

## SqlSession

`SqlSession` 是执行 sql 命令的接口：

```java
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
} finally {
  session.close();
}
```

`SqlSession` 可以执行所有的 sql 命令、做事务提交、回滚、获取 Mapper 实例等，非常强大。`SqlSession` 也是方法作用域级别的，并且必须被正确关闭：

> 每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。**绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。**也绝不能将 SqlSession 实例的引用放在任何类型的管理作用域中，比如 Servlet 架构中的 HttpSession。如果你现在正在使用一种 Web 框架，要考虑 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。换句话说，每次收到的 HTTP 请求，就可以打开一个 SqlSession，返回一个响应，就关闭它。这个关闭操作是很重要的，你应该把这个关闭操作放到 finally 块中以确保每次都能执行关闭。下面的示例就是一个确保 SqlSession 关闭的标准模式：
>
> ```java
> SqlSession session = sqlSessionFactory.openSession();
> try {
>   // do work
> } finally {
>   session.close();
> }
> ```

## Mapper

`Mapper` 是资源和数据库实例的映射，提供了相关操作来做转换。可以用两种方式写：xml 或 annotation。

mapper 实例从 `SqlSession` 获得，所以生命周期和 SqlSession 相同。

### xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

访问：

```java
Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
```

当然，我们希望通过 java 接口来调用这些方法，所以可以写相应的 `mapper` 接口 ，只要保证 `namespace` 是一致的即可。`namespace` 是实现接口绑定的方式。mybatis 基于命名空间的命名解析规则如下：

>* 完全限定名（比如“com.mypackage.MyMapper.selectAllThings”）将被直接查找并且找到即用。
>* 短名称（比如“selectAllThings”）如果全局唯一也可以作为一个单独的引用。如果不唯一，有两个或两个以上的相同名称（比如“com.foo.selectAllThings ”和“com.bar.selectAllThings”），那么使用时就会收到错误报告说短名称是不唯一的，这种情况下就必须使用完全限定名。

一旦绑定了接口，就可以用如下方式访问 mapper 方法了：

```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);
```

### annotation

也可以不依赖于 xml，直接在 mapper 接口上通过 annotation 定义 sql：

```java
package org.mybatis.example;
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  Blog selectBlog(int id);
}
```

annotation 可能更简洁一些，但是 mybatis 目前还是 xml 更强大。大家可以依据需求自由的在两种方式之间切换，mybatis 会自己检测。

>由于 Java 注解的一些限制加之某些 MyBatis 映射的复杂性，XML 映射对于大多数高级映射（比如：嵌套 Join 映射）来说仍然是必须的。有鉴于此，如果存在一个对等的 XML 配置文件的话，MyBatis 会自动查找并加载它（这种情况下， BlogMapper.xml 将会基于类路径和 BlogMapper.class 的类名被加载进来）