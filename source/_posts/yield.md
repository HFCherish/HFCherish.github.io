---
title: 'c# contextual keywords: yield'
toc: true
tags:
  - 'c#'
date: 2020-03-10 11:19:31
---


[yield](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/yield) is a contextual keywords. When it shows in a statement, it means the method or get accessor in which it appears is an iterator. Thus it provides a simple way to define an iterator, rather than a class that implements `IEnumerable` or `IEnumerator`.

> When you use the `yield` [contextual keyword](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/#contextual-keywords) in a statement, you indicate that the method, operator, or `get` accessor in which it appears is an iterator. Using `yield` to define an iterator removes the need for an explicit extra class (the class that holds the state for an enumeration, see [IEnumerator](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerator-1) for an example) when you implement the [IEnumerable](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerable) and [IEnumerator](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerator) pattern for a custom collection type.

## Grammar

```c#
yield return expression;	// return an element in the iterator
yield break;	// end the iterator
```



## Q & A

#### What's an iterator?

An iterator means that it can be looped in `foreach` or LINQ query. In this case, it means the method or get accessor containing `yield` can be consumed by `foreach` or LINQ query.

#### what does `yield` means in this iterator?

The `yield return` will return an element in the iterator. During the loop, the iterator use `MoveNext` to get i (take [power method](#yield-in-method) as an example), and the `MoveNext` stop at the next `yield return` expression, and the `Current` property of the iterator is updated as this value, too.

#### when does the iterator stopped?

* when there's `yield break`
* when the method body is end

#### what's the requirements to define such iterator?

- The return type must be [IEnumerable](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerable), [IEnumerable](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1), [IEnumerator](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerator), or [IEnumerator](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerator-1).
- The declaration can't have any [in](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/in-parameter-modifier) [ref](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref) or [out](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/out-parameter-modifier) parameters.
- Don't use `yield` in [Lambda expressions](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions) and [anonymous methods](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/delegate-operator).
- Don't use `yield` in methods that contain unsafe blocks. For more information, see [unsafe](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/unsafe).
- Don't use `yield return` in a try-catch block. A `yield return` statement can be located in the try block of a try-finally statement.
- `yield break` can be located in a try block or a catch block but not a finally block

# examples

### yield in method<a name = 'yield-in-method' />

```c#
static void Main(string[] args)
{
  	var powers = Power(2, 10);	// won't execute the body of Power
    foreach (var i in powers)
    {
        Console.WriteLine($"{i} ");
    }
}

// =================== yield in method ========================
public static IEnumerable<int> Power(int number, int exponent)
{
    int result = 1;
    for (int i = 0; i < exponent; i++)
    {
        result = result * number;
        yield return result;
    }
}
```

### yield in get accessor<a name = 'yield-in-get-accessor' />

```c#
static void Main(string[] args)
{
    foreach (var i in new Galaxies().AllGalaxies)
    {
        Console.WriteLine($"{i.Name}, {i.Age}");
    }
}

class Galaxies
{
  // =================== yield in get accessor ========================
    public IEnumerable<Galaxy> AllGalaxies
    {
        get
        {
            yield return new Galaxy("The milky way", 1000);
            yield return new Galaxy("Tadpole", 1000);
            yield return new Galaxy("Andromeda", 1000);
        }
    }
}

class Galaxy
{
    public string Name { get; }
    public int Age { get; }

    public Galaxy(string name, int age)
    {
        Name = name;
        Age = age;
    }
}
```

