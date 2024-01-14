---
title: Java线程池
author: 荀乐
tags:
  - 多线程
categories:
  - Java
date: 2022-03-29 01:05:46
---

# 初探线程池

## 为什么要使用线程池

>   使用线程池主要有以下三个原因：
>
>   1.  创建/销毁线程需要消耗系统资源，线程池可以**复用已创建的线程**。
>   2.  **控制并发的数量**。并发数量过多，可能会导致资源消耗过多，从而造成服务器崩溃。（主要原因）
>   3.  **可以对线程做统一管理**。

Java中线程池的顶层接口是`Executor`接口，而`ThreadPoolExecutor`是这个接口的实现类

---

## 线程池核心参数

-   `int corePoolSize`：该线程池中的核心线程数

    >   线程池中有两类线程：核心线程和非核心线程。核心线程

-   `int maximumPoolSize`：该线程池允许的最大线程数

    >   非核心线程数+核心线程数

-   `long keepAliveTime`：非核心线程闲置的超时时长

    >   非核心线程如果处于闲置状态超过该值，就会被销毁。

-   `TimeUnit`：keepAliveTime的时间单位

    >   `TimeUnit`是一个枚举类型

-   `workQueue`：阻塞队列，维护等待执行的线程

-   `threadFactory`：创建线程的工厂

-   `RejectedExecutionHandler`：拒绝策略的处理







---

## ThreadPoolExecutor的策略

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











----

## 线程池处理流程

![img](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153148.png)





---

## 常见线程池

>   `java.util.concurrent.Executors` 类提供了几种静态方法创建线程池
>
>   1.  `newCachedThreadPool`
>   2.  `newFixedThreadPool`
>   3.  `newSingleThreadExecutor`
>   4.  `newScheduledThreadPool`
