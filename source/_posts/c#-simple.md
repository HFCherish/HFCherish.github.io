---
title: 'c# simple'
toc: true
tags:
  - 'c#'
date: 2019-06-05 17:11:28
---


# .net, asp.net, c#

c# is like java language specification;

.net is like jdk/javase/javaee

[asp.net](https://dotnet.microsoft.com/apps/aspnet): is like springboot

1. [default](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/default-value-expressions), as, is
2. sln: solution ——> csproj: c sharp project ——> files [.sln vs .csproj](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/default-value-expressions)

# Key concepts

ref: [c# concepts](https://blog.csdn.net/baidu_32134295/article/details/51285603)

* **solution**: a complete application, similar to maven project. It contains several c# project like frontend, backend, library to compose a complete application.
* **project**: similar to maven module. It can be a web project, a library, a windows program, etc.
* **assembly**: similar to maven jar. A c# project is corresponding to an assembly. An assembly can be a `dll`, `exe`, etc.
* **namespace**: similar to java package. It's a logical concept to avoid naming conflicts while assembly is a physical concept. A namespace can be in different assemblies and an assembly can contains multiple namespaces.

# Accessibility levels

ref: [accessibility levels](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/accessibility-levels)

* **public**: access is not restricted. (all)
* **private**: limited to containing type (only self)
* **protected**: limited to the containing class and types derived from the containing class. (sub-classes)
* **internal**: limited to the current assembly (only the same assembly)
* **protected internal**: limited to the current assembly or types derived from the containing class (same assembly & sub-classes)
* **protected private**: limited to the containing class and types derived from the containing class in the current assembly (sub-classes in same assembly)

# data types

## Nullable

[Nullable<T>](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/nullable-types/) (T?): similar to Optional in java, but the T can only be [value type](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/introduction#types-and-variables), which are simple types, enum types, struct types, and nullable types. Because value type has no null value (not alike reference type — — object), and there're situations when their values are undefined, nullable is born.

```c#
int? x = null    // int? is the shorthand for Nullable<int>
```

## Delegate

[delegate](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/introduction#delegates) is like `@FunctionalInterface` in java.

> A ***delegate type*** represents references to methods with a particular parameter list and return type. Delegates make it possible to treat methods as entities that can be assigned to variables and passed as parameters. 

```c#
delegate double Function(double x);    // functional interface definition

static double[] Apply(double[] a, Function f) {}  // use delegate as a method param

Apply({0.0, 0.5, 1.0}, (double x) -> x*x);   // define an anoymous function
```

# Method

[Method](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/introduction#methods) contains parameters & return type & body & modifier & type parameters.

## parameters

Argument is where the initial value is from, and parameter is used to pass value/references to methods. 

Parameter has modifier (e.g. out, final, this, params) and type

Argument are passed as parameter in 4 ways:

### value parameter: 

parameter change won't affect the argument

1. only for input parameter passing
2. optional by specify default value

### reference parameter

parameter change will affect the argument

1. for input/output parameter passing

```c#
public void swap(ref int x, ref int y)
```

### output parameter

similar to reference parameter except for that the initial value is unimportant.

```c#
public void divide(int x, int y, out int res, out int remainder){}

divide(1,2,out var res, out var remainder);
```

### parameter arrays

similar to java `…`

## extension methods

[extension method](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) is a mechanism that you can "add method" to a class without extending from it.

This method:

1. must be static
2. works in scope when you explictly import the namespace into you source code with a `using` directive.
3. must be the first param of method.
4. when used, it's same as the normal instance method.

```c#
namespace ExtensionMethods
{
    public static class MyExtensions
    {
        public static int WordCount(this String str)
        {
            return str.Split(new char[] { ' ', '.', '?' }, 
                             StringSplitOptions.RemoveEmptyEntries).Length;
        }
    }   
}
```
