---
title: Thread-Safe Solutions
toc: true
date: 2020-07-09 10:37:16
tags:
	- thread
---

# interprocess synchronization (heavy way)

[Monitor](https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor?view=netcore-3.1)

* lock on an object
* use `TryEnter` if no block needed

[Mutex](https://docs.microsoft.com/en-us/dotnet/api/system.threading.mutex?view=netcore-3.1)

* a simple exclusive lock
* Use wait time-out interval to decide the block behavior

> If a thread acquires a mutex, the second thread that wants to acquire that mutex is suspended until the first thread releases the mutex.

[Semaphore](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphore?view=netcore-3.1)

* Limits the number of threads that can access a resource or pool of resources concurrently.
* similar to Mutex, but `Semaphore` can hava multiple threads access the critical resources concurrently.
* different from `Mutex`, `Semaphore` doesn't enfoce thread identity on calls to `WaitOne` and `Release`, that is any thread can release `Semaphore` requested by another thread.