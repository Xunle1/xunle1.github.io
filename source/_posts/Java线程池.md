---
title: Java 线程池
author: 荀乐
tags:
  - 多线程
  - 并发
  - Java
date: 2022-03-29 01:05:46
---

# 为什么要使用线程池

使用线程池主要有以下三个原因：

1.  创建/销毁线程需要消耗系统资源，线程池可以**复用已创建的线程**。
2.  **控制并发的数量**。并发数量过多，可能会导致资源消耗过多，从而造成服务器崩溃。（主要原因）。
3.  可以对线程做统一管理。

Java 中线程池的顶层接口是`Executor`接口，而`ThreadPoolExecutor`是这个接口的实现类。

# 线程池核心参数

1. `int corePoolSize`：该线程池中的核心线程数，线程池中有两类线程：核心线程和非核心线程。
2. `int maximumPoolSize`：该线程池允许的最大线程数 = 非核心线程数 + 核心线程数。
3. `long keepAliveTime`：非核心线程闲置的超时时长，非核心线程如果处于闲置状态超过该值，就会被销毁。
4. `TimeUnit`：keepAliveTime 的时间单位。
5. `workQueue`：阻塞队列，维护等待执行的线程。
6. `threadFactory`：创建线程的工厂。
7. `RejectedExecutionHandler`：拒绝策略处理。

# ThreadPoolExecutor 的策略

线程池本身有一个调度线程，用来管理线程池的各种事务。

线程池也有自己的状态。

```java
// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

# 线程池处理流程

![img](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153148.png)

# 常见线程池

`java.util.concurrent.Executors` 类提供了几种静态方法创建线程池

1. `newCachedThreadPool`
2. `newFixedThreadPool`
3. `newSingleThreadExecutor`
4. `newScheduledThreadPool`

# 阻塞队列

在**生产者-消费者**模型中，生产者负责生产资源，消费者负责消费资源，然而有时会出现生产者和消费者生产消费的速率不匹配的情况，这种情况下会导致资源池为空或为满。当资源池为空时，需要阻塞消费者，唤醒生产者；当资源池为满时，需要阻塞生产者，唤醒消费者。

JDK 中的阻塞队列（BlockingQueue）可以帮助完成这个任务，使用者只需要向队列中添加或取出资源，而无需关心共享资源的并发访问问题和阻塞唤醒流程。

**BlockingQueue** 是 `java.util.concurrent` 包下的接口，有多种实现类提供了不同的存取方法，和普通的队列不同的是都支持多线程环境。

## 操作方法

BlockingQueue 提供了四组不同的方法操作队列中的元素。

1. 抛出异常：`add(e)`、`remove()`、`element()`
2. 返回特定值（`null` 或者 `false`）：`offer(e)`、`poll()`、`peek()`
3. 阻塞当前线程：`put(e)`、`take()`
4. 指定唤醒时间：`offer(e, time, unit)`、`pool(time, unit)`

## 实现类

### ArrayBlockingQueue

由**数组**组成的有界阻塞队列。

### LinkedBlockingQueue

由**链表**组成的阻塞队列。默认大小为 `Integer.MAX_VALUE`。

### DelayQueue

由实现了 `Delay` 接口的对象组成的无界阻塞队列，传入的对象只有当延迟时间到期后才可以被取出。

### PriorityBlockingQueue

优先阻塞队列，同样也是无界的，传入的对象通过实现 `Comparator` 接口排序。因为是无界队列所以有引发 OOM 的风险。

### SynchronousQueue

这个队列没有任何内部容量，每个 put 必须等待一个 take 操作，反之亦然。
