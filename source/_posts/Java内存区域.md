---
title: Java 内存区域
author: 荀乐
categories: 技术
tags:
  - 内存区域
  - JVM
date: 2021-09-07 22:46:10
---

# Java内存区域

**JDK 1.8以前：**

![JVM运行时数据区域](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153655.png)

**JDK 1.8以后:**

![Java运行时数据区域JDK1.8](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153701.png)

**线程私有：**

-   **虚拟机栈**

    ![虚拟机栈](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153704.jpeg)

    Java 虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息。

    **局部变量表主要存放了编译期可知的各种数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

    **Java 虚拟机栈会出现两种错误：`StackOverFlowError` 和 `OutOfMemoryError`。**

    -   **`StackOverFlowError`：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
    -   **`OutOfMemoryError`：** Java 虚拟机栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出`OutOfMemoryError`异常。

-   **本地方法栈**

    本地方法栈与虚拟机栈发挥的作用相似，区别在于虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈为虚拟机使用到的本地（Native）方法服务

-   **程序计数器**

    程序计数器是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。

    程序计数器的作用：

    1.  字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
    2.  在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

    程序计数器是唯一一个没有规定任何`OutOfMemoryError`的内存区域

**线程共享：**

-   **堆**

    Java堆的目的是存放对象实例，几乎所有对象实例都在堆上分配内存。

    Java堆也是垃圾收集器管理的主要区域。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代；再细致一点有：`Eden` 空间、`From Survivor`、`To Survivor` 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

    JDK 7及以前堆内存区域：

    ![JVM堆内存结构-JDK7](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153708.png)

    JDK 8以后堆内存区域：

    ![JVM对内存区域-jdk8](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153711.png)

-   **方法区**

    方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

-   **运行时常量池**

    运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池表（用于存放编译期生成的各种字面量和符号引用）

-   **直接内存**

    直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 `OutOfMemoryError` 错误出现。



---

*参考阅读：*

[深入理解Java虚拟机（第3版） (豆瓣) (douban.com)](https://book.douban.com/subject/34907497/)

[JavaGuide](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/jvm/Java内存区域.md)