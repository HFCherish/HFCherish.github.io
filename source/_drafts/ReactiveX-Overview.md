---
title: ReactiveX Overview
toc: true
date: 2020-03-13 10:19:55
tags:
	- reactivex
---



When an observer subscribes to an observable sequence, the thread calling the Subscribe method can be different from the thread in which the sequence runs till completion. Therefore, the Subscribe call is asynchronous in that **the caller is not blocked until the observation of the sequence completes**. This will be covered in more details in the [Using Schedulers](https://docs.microsoft.com/en-us/previous-versions/dotnet/reactive-extensions/hh242963(v%3dvs.103)) topic.

```

thread 1: -- == ** ^^
thread 2:  --
```



 The Publish operator provides a mechanism to share **subscriptions** by broadcasting a single subscription to multiple subscribers. ====> ours is cold



Being **free-threaded** means that you are not restricted to which thread you choose to do your work. For example, you can choose to do your work such as **invoking a subscription, observing or producing notifications, on any thread** you like. 

 if you do not introduce any scheduling, your callbacks will be invoked on the same thread that the *OnNext*/*OnError*/*OnCompleted* methods are invoked from.



source: asynchronous data stream ===> Observable

operator: ReactiveX provides [a collection of operators](http://reactivex.io/documentation/operators.html) with which you can filter, select, transform, combine, and compose Observables. This allows for efficient execution and composition.



**You can think of the Observable class as a “push” equivalent to [Iterable](http://docs.oracle.com/javase/7/docs/api/java/lang/Iterable.html), which is a “pull.”** With an Iterable, the consumer pulls values from the producer and the thread blocks until those values arrive. By contrast, with an Observable the producer pushes values to the consumer whenever values are available. This approach is more flexible, because values can arrive synchronously or asynchronously.

**Observables fill the gap by being the ideal way to access asynchronous sequences of multiple items**

|              | single items       | multiple items         |
| :----------- | :----------------- | ---------------------- |
| synchronous  | `T getData()`      | `Iterable getData()`   |
| asynchronous | `Future getData()` | `Observable getData()` |

**An Observable is the asynchronous/push** [“dual”](http://en.wikipedia.org/wiki/Dual_(category_theory)) **to the synchronous/pull Iterable**

| event          | Iterable (pull)    | Observable (push)    |
| :------------- | :----------------- | :------------------- |
| retrieve data  | `T next()`         | `onNext(T)`          |
| discover error | throws `Exception` | `onError(Exception)` |
| complete       | `!hasNext()`       | `onCompleted()`      |





The Observable type adds two missing semantics to [the Gang of Four’s Observer pattern](http://en.wikipedia.org/wiki/Observer_pattern), to match those that are available in the Iterable type: (Since it's a push lterable)

1. **the ability for the producer to signal to the consumer that there is no more data available** (a foreach loop on an Iterable completes and returns normally in such a case; an Observable calls its observer’s `onCompleted` method)
2. **the ability for the producer to signal to the consumer that an error has occurred** (an Iterable throws an exception if an error takes place during iteration; an Observable calls its observer’s `onError` method)





# “Hot” and “Cold” Observables

When does an Observable begin emitting its sequence of items? It depends on the Observable. A “hot” Observable may begin emitting items as soon as it is created, and so any observer who later subscribes to that Observable may start observing the sequence somewhere in the middle. A “cold” Observable, on the other hand, waits until an observer subscribes to it before it begins to emit items, and so such an observer is guaranteed to see the whole sequence from the beginning.

In some implementations of ReactiveX, there is also something called a “Connectable” Observable. Such an Observable does not begin emitting items until its [Connect](http://reactivex.io/documentation/operators/connect.html) method is called, whether or not any observers have subscribed to it.