---
title: Java代理模式
author: 荀乐
tags:
  - proxy
categories:
  - 设计模式
date: 2021-11-03 21:41:20
---
# 代理模式

代理通常指的通过一个代理类实现对真实类的控制访问。

在有些情况下，一个客户不能或者不想直接访问另一个对象，这时需要找一个中介帮忙完成某项任务，这个中介就是代理对象。比如租房子时与中介沟通租房的一切事宜，不直接接触房东。

Java中根据代理的创建时期，代理的类型分为两种。

-   静态代理：程序员创建代理类或特定工具类源码对其编译，在程序运行前就生成了.class文件
-   动态代理：在程序运行时通过反射机制动态创建

# 静态代理

创建一个`RentService`接口

```java
public interface RentService {
    
    void pay();
}
```

实现类

```java
public class RentServiceImpl implements RentService{

    @Override
    public void pay() {
        System.out.println("房客付钱咯...");
    }
}
```

代理类

```java
public class RentServiceProxy implements RentService{
    private RentService rentService;

    public RentServiceProxy(RentService rentService) {
        this.rentService = rentService;
    }

    public void see() {
        System.out.println("房客看房咯...");
    }

    @Override
    public void pay() {
        see();
        rentService.pay();
        sign();
    }

    public void sign() {
        System.out.println("房客签名咯...");
    }
}
```

测试类

```java
public class TestProxy {
    public static void main(String[] args) {
        RentService rentService = new RentServiceProxy(new RentServiceImpl());
        rentService.pay();
    }
}
```

结果如图

![静态代理](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206154028.png)

通过`RentServiceProxy`实现了对`RentService`类的拓展，在测试类中代码量没有增加但是扩展了功能。

但静态代理的缺点在于：

-   当需要代理多个类的时候，由于代理对象要实现与目标对象一致的接口，有两种方式：
    -   只维护一个代理类，由这个代理类实现多个接口，但是这样就导致**代理类过于庞大**。
    -   新建多个代理类，每个目标对象对应一个代理类，但是这样会**产生过多的代理类。
-   当接口需要增加、删除、修改方法的时候，目标对象与代理类都要同时修改，**不易维护**。

# 动态代理

Java中动态代理有两种实现方法：

-   JDK动态代理

涉及两个类 `java.lang.reflect.Proxy` 和 `java.lang.reflect.InvocationHandler`。

`Proxy` 提供了创建动态代理类和实例的静态方法，它也是由这些方法创建的所有动态代理类的超类。

`InvocationHandler` 是由代理实例的调用处理程序实现的接口。

JDK 动态代理的对象必须实现一个或多个接口。如果想代理没有实现接口的类，就可以使用CGLIB实现。JDK 动态代理之所以**只能代理接口**是因为**代理类本身已经 extends 了 Proxy，而java是不允许多重继承的**，但是允许实现多个接口。

-   Cglib动态代理

使用cglib代理的对象则无需实现接口，可以在运行时动态的生成某个类的子类，故需要某个类不能被标记为final

## JDK动态代理

使用`Proxy.newProxyInstance`方法生成代理对象

```java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
```

-   `loader` 为代理类的类加载器
-   `interfaces` 为代理类的接口数组
-   `h` 为方法调用处理器

使用 `InvocationHandler` 的 `invoke` 方法实现方法调用

```java
 public Object invoke(Object proxy, Method method, Object[] args)
```

-   `proxy`：触发方法的代理类实例
-   `method`：代理类实现的接口中的方法
-   `args`：方法参数

实现一个Handler，继承 `InvocationHandler`

```java
public class DynamicProxyHandler implements InvocationHandler {
    private RentService rentService;

    public DynamicProxyHandler(RentService rentService) {
        this.rentService = rentService;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("房客看房咯...");
        Object o = method.invoke(rentService, args);
        System.out.println("房客签名咯...");
        return o;
    }
}
```

测试类

```java
public class TestProxy {
    public static void main(String[] args) {
        
        RentService target = new RentServiceImpl();
        //自定义方法调用处理器
        DynamicProxyHandler handler = new DynamicProxyHandler(target);
        //获得代理类实例
        RentService rentServiceProxy = (RentService) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                handler);
        rentServiceProxy.pay();
    }
}
```

结果和静态代理类相同。

## [CGLIB](https://github.com/cglib/cglib)动态代理

 JDK中提供的生成动态代理类的机制有个鲜明的特点是： 某个类必须有实现的接口，而生成的代理类也只能代理某个类接口定义的方法。比如上面例子中如果`RentServiceImpl`有其他方法`live()`，JDK动态代理类中就没有这个方法。

而`CGLIB(Code Generation Library)`没有接口也能实现动态代理

**示例代码**

自定义`Interceptor`

```java
public class RentInterceptor implements MethodInterceptor {

    /**
     *
     * @param object 增强的对象
     * @param method 拦截的方法
     * @param args 方法参数
     * @param methodProxy 对方法的代理，invokeSuper方法表示对被代理对象方法的调用
     * @return 执行结果
     * @throws Throwable
     */
    @Override
    public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("房客看房咯...");
        Object result = methodProxy.invokeSuper(object, args);
        System.out.println("房客签名咯...");
        return result;
    }
}
```

测试

```java
public class TestProxy {
    public static void main(String[] args) {
        
        RentInterceptor interceptor = new RentInterceptor();
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(RentServiceImpl.class);
        enhancer.setCallback(interceptor);
		
        //创建代理类，可以对类进行代理
        RentServiceImpl proxy = (RentServiceImpl)enhancer.create();
        System.out.println("cglib: " + proxy.getClass());
        proxy.pay();
        proxy.decorate();
    }
}
```

运行结果

![cglib](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206154032.png)

