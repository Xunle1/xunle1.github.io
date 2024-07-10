---
title: JavaIO
author: 荀乐
tags:
  - IO
  - Java
date: 2022-03-29 01:11:43
---
# JavaNIO

## 概述

### 三种IO模型

Java支持三种`I/O`模式

-   `BIO`
-   `NIO`
-   `AIO`

**BIO (Blocking):**

>   JDK1.4以前的同步阻塞IO，当客户端有连接请求时，服务器就需要启动一个线程进行处理。
>
>   ![img](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153351.png)

---

**NIO (Non-blocking)**

>   JDK1.4以后，同步非阻塞IO，服务端使用一个线程处理多个请求，客户端发送的连接请求会注册到多路复用器上，多路复用器轮询到连接有`I/O`请求就处理
>
>   ![img](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153354.png)

---

**AIO (Asynchronous)**

>   JDK1.7以后，异步非阻塞IO。它的特点是先由操作系统完成后才通知服务端程序启动线程去处理。



---

**三种IO模型的适用场景**

-   `BIO` 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，程序简单易理解。
-   `NIO` 方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，弹幕系统，服务器间通讯等，编程比较复杂。
-   `AIO` 方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用 `OS` 参与并发操作，编程比较复杂。



---

**`NIO`和`BIO`的比较**

1.  `BIO` 以流的方式处理数据，而 `NIO` 以块的方式处理数据，块 `I/O` 的效率比流 `I/O` 高很多。
2.  `BIO` 是阻塞的，`NIO` 则是非阻塞的。
3.  `BIO` 基于字节流和字符流进行操作，而 `NIO` 基于 `Channel`（通道）和 `Buffer`（缓冲区）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。`Selector`（选择器）用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道。



---

## NIO原理

### NIO三大核心

`NIO` 有三大核心部分: **`Channel`（通道）、`Buffer`（缓冲区）、`Selector`（选择器）** 。

**NIO三大核心关系图**

<img src="https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153401.png" alt="img" style="zoom: 67%;" />

1.  每个 `Channel` 都会对应一个 `Buffer`。
2.  `Selector` 对应一个线程，一个线程对应多个 `Channel`（连接）。
3.  该图反应了有三个 `Channel` 注册到该 `Selector` //程序
4.  程序切换到哪个 `Channel` 是由事件决定的，`Event` 就是一个重要的概念。
5.  `Selector` 会根据不同的事件，在各个通道上切换。
6.  `Buffer` 就是一个内存块，底层是有一个数组。
7.  `Buffer`可以通过`flip`方法切换读写模式。



---

### 缓冲区（Buffer）

缓冲区（`Buffer`）：缓冲区本质上是一个**可以读写数据的内存块**，可以理解成是一个**容器对象（含数组）**，该对象提供了一组方法，可以更轻松地使用内存块，，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。`Channel` 提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由 `Buffer`

![img](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153409.png)

**Buffer类及其子类**

![image-20210726001826565](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153414.png)

<img src="https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153417.png" alt="image-20210726001849519" style="zoom: 67%;" />





---

### 通道（Channel）

1.  `NIO`的通道类似于流，但有些区别如下：
    -   通道可以同时进行读写，而流只能读或者只能写
    -   通道可以实现异步读写数据
    -   通道可以从缓冲读数据，也可以写数据到缓冲:
2.  `BIO` 中的 `Stream` 是单向的，例如 `FileInputStream` 对象只能进行读取数据的操作，而 `NIO` 中的通道（`Channel`）是双向的，可以读操作，也可以写操作。
3.  `Channel` 在 `NIO` 中是一个接口 `public interface Channel extends Closeable{}`
4.  常用的 `Channel` 类有：`FileChannel`、`DatagramChannel`、`ServerSocketChannel` 和 `SocketChannel`。`ServerSocketChanne` 类似 `ServerSocket`、`SocketChannel` 类似 `Socket`
5.  `FileChannel` 用于文件的数据读写，`DatagramChannel` 用于 `UDP` 的数据读写，`ServerSocketChannel` 和 `SocketChannel` 用于 `TCP` 的数据读写。h
