---
title: java reflection
toc: true
tags:
  - java
date: 2018-07-14 22:37:43
---



# 泛型
java 泛型存在类型擦除（参见 [java 泛型](https://blog.csdn.net/briblue/article/details/76736356)）

```java
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();

System.out.println(l1.getClass() == l2.getClass());	// return true, 两个都是 List.class
```

## 获取运行时泛型类型

类型擦除使得根据类定义获取 runtime 泛型类型是不可能的，一般有几种方法(参见 [stackoverflow](https://stackoverflow.com/questions/3403909/get-generic-type-of-class-at-runtime))：

1. 根据类对象实例获取，可参见 [handle java generic types with reflection](http://qussay.com/2013/09/28/handling-java-generic-types-with-reflection/#has_default_constructor)
	* eg. `Class<T> tClass = (Class<T>) ReflectionUtil.getClass(ReflectionUtil.getParameterizedTypes(this)[0]);`
2. 从父类中获取（要求父类有相同的泛型参数）
	* eg. `Class<T> tClass = (Class<T>) ((ParameterizedType) getClass().getGenericSuperclass()).getActualTypeArguments()[0]`
3. 通过方法存储泛型类型为 field。但这意味着所有的 client 都必须要通过相应方法设置该 field

```java
// 通过方法（constructor）存储泛型类型
public class GenericClass<T> {

     private final Class<T> type;

     public GenericClass(Class<T> type) {
          this.type = type;
     }

     public Class<T> getMyType() {
         return this.type;
     }
}
```
## 获取一个带泛型信息的 class 变量

例如当使用 `new ObjectMapper().readValue(string, someClass)`，而 someClass 是包含泛型参数的类型（eg. List<Integer>），如何获取这样的 class 变量？

参见 [stackOverflow](https://stackoverflow.com/a/6349488/10003123)，总结如下：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
ObjectMapper mapper = new ObjectMapper();

// as Array
MyClass[] myObjects = mapper.readValue(json, MyClass[].class);

// as List
List<MyClass> myObjects = mapper.readValue(jsonInput, new TypeReference<List<MyClass>>(){});

// as List (another more generic way)
List<MyClass> myObjects = mapper.readValue(jsonInput, mapper.getTypeFactory().constructCollectionType(List.class, MyClass.class));

// as List (using TypeToken in Gson)
Class<List<MyClass>> tClass = new TypeToken<List<MyClass>() {
        }.getRawType();
        
// use c
```