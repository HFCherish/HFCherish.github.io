---
title: java stream parallel 有时比 sequential 还慢？
tags:
  - java
---

# 为什么 java stream parallel 有时比 sequential 执行还慢？
## 场景
考虑下边的代码，并行执行不一定比顺序执行快，甚至很多时候都是更慢的。

```java
@Test
    public void should_not_sure_if_without_warm_up() {
        String[] array = new String[1000000];
        Arrays.fill(array, "AbabagalamagA");
        System.out.println("Benchmark...");
        for (int i = 0; i < 5; ++i) {
            System.out.printf("Run %d:  sequential %s  -  parallel %s\n",
                    i,
                    test(() -> sequential(array)),
                    test(() -> parallel(array)));
        }

    }

    private static void sequential(String[] array) {
        Arrays.stream(array).map(String::toLowerCase).collect(Collectors.toList());
    }
    private static void parallel(String[] array) {
        Arrays.stream(array).parallel().map(String::toLowerCase).collect(Collectors.toList());
    }
    private static String test(Runnable runnable) {
        long start = System.currentTimeMillis();
        runnable.run();
        long elapsed = System.currentTimeMillis() - start;
        return String.format("%4.2fs", elapsed / 1000.0);
    }
```
## 为什么？
有几个原因（[stackoverflow](http://www.theserverside.com/definition/just-in-time-compiler-JIT)）：

1. **stream 的并行执行比串行执行要做更多的事**。并行执行需要拆分程序，使得程序可以并行执行，最后要合并结果。例如，上述并行执行涉及到 new 线程池、分配线程执行特定的 string 操作并加到一个 list、最终合并 list。这个程序本身已经执行很快，此时，这些额外开销比本身执行的时间可能还要长，就影响了它最终带来的性能。
2. **编译器、jvm、GC 等会影响代码执行效率，因此对 java 做这些基准测试很微妙**。例如 [JIT compiler](http://www.theserverside.com/definition/just-in-time-compiler-JIT)、GC 等就会很大程度的影响测试结果。
    3. 测试很大程度受 JIT compiler 执行的影响
        4. 在 JIT compiler 完成之前，可能测试已经跑完了。此时顺序执行和并行执行哪个 JIT compiler 先跑完，可能测试就会跑的更快一些
        5. 而且 JIT compiler 什么时候开始跑也不确定。
        6. 并且 JIT compiler 会做一些运行时优化，比如有些代码，其输出没有在任何地方被使用，JIT compiler 会直接消除这些代码的执行。这种情况还是非常容易发生的。此时，你这些测试衡量就更微妙了，因为可能最终执行的测试并不是你所写的测试，而是优化之后的。
        6. 如果在测试执行之前，加上一些预热，就可以保证程序都已经再编译完成，此时评估的就是同等条件下的程序执行效率了（参见下边的 code）。
    4. GC 会影响执行效率，不同的代码会产生不同的 eliminated objects
        5. stream、并行运行等会涉及到很多中间变量的构建、copy 等，比如中间 string、list 等，这时 GC 执行工作量就比较大，会影响最终的测试执行时间，使得测试结果也不可信。

对 java 做这些基准测试，有时结果会比较 confusing，所以建议采用专门的 benchmark 框架来做基准测试，比如 [JMH](http://openjdk.java.net/projects/code-tools/jmh/)，这框架执行过程中，可以看到很多 java 额外执行的一些操作时间等，就可以更好的观察测试结果了。        

```java
    /**
    * 更改测试，加上预热，保证 JIT 编译已完成，此时基本是在同等条件下测试，测试结果相对更可信一些
    */
    @Test
    public void should_parallel_faster_if_has_warm_up() {
        String[] array = new String[1000000];
        Arrays.fill(array, "AbabagalamagA");
        System.out.println("Warmup...");
        for (int i = 0; i < 100; ++i) {
            sequential(array);
            parallel(array);
        }
        System.out.println("Benchmark...");
        for (int i = 0; i < 5; ++i) {
            System.out.printf("Run %d:  sequential %s  -  parallel %s\n",
                    i,
                    test(() -> sequential(array)),
                    test(() -> parallel(array)));
        }
    }

```

### 什么是 [JIT compiler](http://www.theserverside.com/definition/just-in-time-compiler-JIT)
JIT (just-in-time) compiler 指在运行时执行的编译器。

(1) java 是编译成字节码，然后在运行时解释执行的

c、C++ 等编程语言都是直接编译成机器码，可以在机器上直接执行的。但是不同平台处理器有差异，导致用户可能需要为不同平台写多套程序。

java 就提出了 JVM，将代码一次编译成字节码，然后提供不同的 JVM，JVM 会将字节码解释执行为可运行的机器码。

但是解释执行是一行一行做的，就影响了执行效率。这也是为啥 c++ 等会诟病 java 很慢的原因。

(2) 为了提高解释执行的效率，使用了 JIT compiler

正如上文所说，因为解释执行慢，所以在程序运行起来后，同时会执行 JIT compiler，将字节码编译成可执行代码（相当于二次编译）。这就可以一定程度的加快解释执行的效率。而且 JIT compiler 因为可以获取运行时环境、参数等，所以可以做更多的优化


# parallel 慎用？？？
[DZone: parallel 慎用](https://dzone.com/articles/think-twice-using-java-8) 说因为 stream 公用线程池，一个 broken thread 会影响所有 healthy 线程的执行，所以要慎用。

简单看了一些，比如这个 [stackoverflow](https://stackoverflow.com/questions/20375176/should-i-always-use-a-parallel-stream-when-possible)，应该是说 stream 提供了方便的形式去写 function、可读性高、promote 大家写出 side-effects-free 的代码，但是 stream 本身还是有很多缺陷的。


公用线程池的测试代码如下：

```java
@Test
public void should_be_influenced_by_long_tasks() throws InterruptedException {
    /** Simulating multiple threads in the system
    * if one of them is executing a long-running task.
    * Some of the other threads/tasks are waiting
    */
    int MAX = 12;
    ExecutorService es = Executors.newCachedThreadPool();

    // 这个线程执行很慢，但是因为共享线程池，因此会影响其他线程的执行。极端情况，这里是一个 broken tread，其他 healthy thread 都会受影响
    es.execute(() -> countPrimes(MAX, 1000));

    // 执行结果不确定，因为有上边的长线程。如果注释掉上边线程，下边这个可以很快执行
    es.execute(() -> countPrimes(MAX, 0));
    es.execute(() -> countPrimes(MAX, 0));
    es.execute(() -> countPrimes(MAX, 0));
    es.execute(() -> countPrimes(MAX, 0));
    es.execute(() -> countPrimes(MAX, 0));
    es.shutdown();
    es.awaitTermination(60, TimeUnit.SECONDS);
}

private void countPrimes(int max, int delay) {
    System.out.println(Thread.currentThread().getId() + ": " + range(1, max).parallel().filter(this::isPrime).peek(i -> {
        try {
            sleep(delay);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).count());
}


private boolean isPrime(long n) {
    return n > 1 && rangeClosed(2, (long) sqrt(n)).noneMatch(divisor -> n % divisor == 0);
}

```

## [ForkJoinPool](http://tutorials.jenkov.com/java-util-concurrent/java-fork-and-join-forkjoinpool.html)
[这个文章](https://www.jianshu.com/p/bd825cb89e00) 介绍了 ForkJoinPool，说是 parallel stream 实现的主要原理和背后手段

stream 的并发执行现在基本上都是采用分治法，先拆分用多线程逐个处理，然后再合并结果。最后的合并操作必须在前边某几个线程执行完之后才做。

而普通的线程池 [ThreadPoolExecutor]() 就是构建一个线程池，并发执行，但是它没办法决定线程执行的父子关系。

ForkJoinPool 就是为了解决上述问题而存在，它可以让子任务并发执行完成之后，才开始执行父任务。除此以外，和 ThreadPoolExecutor 一样，都是用一个无限队列来保存待执行的任务。

ForkJoinPool 采用了一个通用线程池，实现了 **[工作窃取](http://ifeve.com/talk-concurrency-forkjoin/)**。工作窃取指某个线程从其他队列里窃取任务来执行。ForkJoinPool 就可以？？？？？

# 什么时候用 parallel
目前来说，在 java 中：

1. 如果是数据量很大的操作，可以考虑用 parallel
3. 如果有性能问题，再考虑用 parallel
5. 如果确实有多核，再考虑用
6. 如果确实是无 side effect 的函数，才可以考虑用
4. 如果已经有其他并行措施，可以不用 parallel
2. 如果数据操作很慢，慎用（可能 block 其他 thread）
3. 如果数据操作很快，也慎用（可能这个时候用并行的额外开销会超过它所能带来的优势）

[这篇文章](https://blog.oio.de/2016/01/22/parallel-stream-processing-in-java-8-performance-of-sequential-vs-parallel-stream-processing/)也对比了并行和串行 stream，然后画了个决策象限图，如下图所示：

![parallel 决策象限图](https://blog.oio.de/wp-content/uploads/2016/01/stream_performance_image3.png)

跟上边类似，关注下边四个方面：

1. `number_of_elements * cost_per_element` 比较大。这可以比较好的解决这种状况：每个元素运行很快时，如果数据量大就可以用；如果每个元素运行稍费时些，即使数据量不那么大，也 ok。但是应该要避免过于费时的那些场景，见上边的分析。
1. source collection 可以很高效的被拆分（这样才方便拆线程处理）
2. 每个元素的函数执行是独立的（这才可以并行处理，即并行首先要求 side effect free）
3. 多核
