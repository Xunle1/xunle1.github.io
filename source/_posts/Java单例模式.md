---
title: Java 单例模式
author: 荀乐
tags:
  - singleton
  - 设计模式
date: 2021-11-04 22:13:24
---
# 单例模式

**单例（Singleton）模式**：指一个类只有一个实例，且该类能自行创建这个实例的一种模式。

单例模式有 3 个特点：

1.  单例类只有一个实例对象；
2.  该单例对象必须由单例类自行创建；
3.  单例类对外提供一个访问该单例的全局访问点。

对于Java来说，通过将构造方法私有化使外部无法调用构造方法生成多个实例，提供一个静态函数供外部访问单例。单例模式可以保证在一个 JVM 中只存在单一实例。

单例模式通常有两种实现方式

-   懒汉式
-   饿汉式



---

## 饿汉式单例

类一旦加载就创建一个实例，保证在调用`getInstance()`方法前已经创建实例

**参考代码**

```java
public class HungrySingleton {
    private static final HungrySingleton instance = new HungrySingleton();

    private HungrySingleton() {}

    public static HungrySingleton getInstance() {
        return instance;
    }
}
```

## 懒汉式单例

加载时没有生成实例，只有第一次调用时才创建实例。

**参考代码**

```java
public class LazySingleton {
    private static LazySingleton instance = null;

    private LazySingleton() {}

    public static LazySingleton getInstance() {
        if (instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }
}
```

懒汉式对于实例化类呈懒惰态度，即实例化的时机在第一被引用时触发，在类进行加载时，不会将自己实例化(延迟加载技术)。

## 懒汉式单例的并发问题

上述**懒汉式单例**代码存在并发问题，如果多个线程同时访问`LazySingleton.getInstance()`可能导致类被实例化多次。而饿汉式调用`getInstance()`前就已经创建实例，不存在并发问题。

**并发测试**：三个线程调用`LazySingleton.getInstance()`方法，三个线程调用`HungrySingleton.getInstance()`方法

```java
public class TestSingleton {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            LazySingleton instance = LazySingleton.getInstance();
            System.out.println(Thread.currentThread().getName() + "->" + instance);
        }, "t1");

        Thread t2 = new Thread(() -> {
            LazySingleton instance = LazySingleton.getInstance();
            System.out.println(Thread.currentThread().getName() + "->" + instance);
        }, "t2");

        Thread t3 = new Thread(() -> {
            LazySingleton instance = LazySingleton.getInstance();
            System.out.println(Thread.currentThread().getName() + "->" + instance);
        }, "t3");

        t1.start();
        t2.start();
        t3.start();

        Thread t4 = new Thread(() -> {
            HungrySingleton instance = HungrySingleton.getInstance();
            System.out.println(Thread.currentThread().getName() + "->" + instance);
        }, "t4");

        Thread t5 = new Thread(() -> {
            HungrySingleton instance = HungrySingleton.getInstance();
            System.out.println(Thread.currentThread().getName() + "->" + instance);
        }, "t5");

        Thread t6 = new Thread(() -> {
            HungrySingleton instance = HungrySingleton.getInstance();
            System.out.println(Thread.currentThread().getName() + "->" + instance);
        }, "t6");

        t4.start();
        t5.start();
        t6.start();
    }
}
```

**测试结果：**

![并发测试结果](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153618.png)

懒汉式获取单例时产生了多个实例，饿汉式获取单例时没有影响。



---

## 解决线程安全问题

### 对方法加锁

对`getInstance()`方法加锁使同一时间只有一个线程获得单例

```java
public class ThreadSafeSingleton {
    private static volatile ThreadSafeSingleton instance;

    private ThreadSafeSingleton() {}

    public static synchronized ThreadSafeSingleton getInstance() {
        if (instance == null) {
            instance = new ThreadSafeSingleton();
        }
        return instance;
    }
}
```

这样可以保证只会创建一个实例，`volatile`防止指令重排序，但`synchronized`锁住了整个方法，多线程访问时性能会降低。

### 双重检查锁

```java
public class DoubleCheckSingleton {
    private static volatile DoubleCheckSingleton instance;

    private DoubleCheckSingleton() {}

    public static DoubleCheckSingleton getInstance() {
        if (instance == null) {
            synchronized (DoubleCheckSingleton.class) {
                if (instance == null) {
                    System.out.println(Thread.currentThread().getName()+ "正在初始化");
                    instance = new DoubleCheckSingleton();
                }
            }
        }
        return instance;
    }
}
```

如果多个线程进入`getInstance()`方法

-   先判断是否已存在实例，若已存在则直接返回，否则获得锁创建实例
-   线程获得锁后先判断是否已存在实例。原因是如果线程A和线程B同时竞争锁，线程A获得锁进入并创建实例后释放锁，此时线程B获得锁且并不知道实例已被创建的话，会继续创建新实例，所以需要第二次判断。

### 内部类



### 枚举