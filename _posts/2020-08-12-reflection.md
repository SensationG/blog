---
layout: post
title: Java反射基础
date: 2020-08-12
Author: hhw
toc: true
comments: true
tags:  [java]
---



先给出一个最常用的反射例子：

```java
Class.forName("com.mysql.jdbc.Driver");  //注册数据库驱动
```

这里是使用JDBC前使用<u>反射</u>注册mysql驱动。

​		之前只是对 Java反射略有耳闻，但实际上并不清楚为什么需要使用反射以及何时需要使用反射，而Spring内部使用了大量的反射机制，那么如果我们想深入学习Spring，那么首先必须对 Java的反射机制有所了解。因此，这篇文章主要入门地阐述 <u>反射是什么，以及反射如何使用</u>。



## 一、反射是什么

​		一般情况下，我们使用某个类时必定知道它是什么类，是用来做什么的。于是我们直接对这个类进行实例化，之后使用这个类对象进行操作。

```java
Apple apple = new Apple(); //直接初始化，「正射」
apple.setPrice(4);
```

在没有用到反射相关的类的时候，我们都是在做正射，就如上面所示。

​		而反射则是一开始并不知道我要初始化的类对象是什么，自然也无法使用 new 关键字来创建对象了。

这时候，我们使用 JDK 提供的反射 API 进行反射调用：

```java
Class clz = Class.forName("com.chenshuyi.reflect.Apple");
Method method = clz.getMethod("setPrice", int.class);
Constructor constructor = clz.getConstructor();
Object object = constructor.newInstance();
method.invoke(object, 4);
```

上面两段代码的执行结果，其实是完全一样的。但是其思路完全不一样，第一段代码在未运行时就已经确定了要运行的类（Apple），而第二段代码则是在运行时通过<u>字符串</u>值才得知要运行的类（com.chenshuyi.reflect.Apple）。

所以说什么是反射？

**反射就是 当在运行状态中才知道要操作的类是什么时，可以使用反射在运行时获取类的完整构造，并调用对应的方法。**

#### 反射的定义

Java反射机制是在**运行状态中**，对于任意一个类，都能知道这个类的属性和方法。对于任意一个对象，都能调用它的任意一个方法和属性。这种动态获取的信息以及动态调用对象的方法的功能称为java语言的**反射机制**。

#### 反射提供的功能

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时调用任意一个对象的方法

#### 简单案例

```java
public class Apple {

    private String kind;

    public String getKind() {
        return kind;
    }

    public void setKind(String kind) {
        this.kind = kind;
    }

    public static void main(String[] args) throws Exception {
        // 正射
        Apple apple = new Apple();
        apple.setKind("red");
        System.out.println(apple.getKind());
        // 反射
        Class clz = Class.forName("reflection.Apple");
        Constructor appleConstructor = clz.getConstructor();
        Object appleObj = appleConstructor.newInstance();
        Method setKindMethod = clz.getMethod("setKind", String.class);
        setKindMethod.invoke(appleObj, "red");
        Method getKindMethod = clz.getMethod("getKind");
        System.out.println(getKindMethod.invoke(appleObj));
    }
}
```

上面通过正射和反射两种方法进行设值和获取值，结果均相同。

接下来对上面反射的步骤进行解析：

- 首先获取Class对象实例

```java
Class clz = Class.forName("reflection.Apple"); // 输入类的全路径名
```

- 根据Class对象实例获取Constructor对象

```java
Constructor appleConstructor = clz.getConstructor(); // 默认获取无参构造
```

- 通过Constructor对象的newInstance方法获取反射对象的实例

```java
Object appleObj = appleConstructor.newInstance();
```

- 使用Class对象的getMethod方法获取Method对象

```java
Method setKindMethod = clz.getMethod("setKind", String.class); // 方法名，参数类型
```

- 使用Method对象的invoke方法调用目标方法，需传入反射对象实例以及方法参数

```java
setKindMethod.invoke(appleObj, "red"); // 反射实例，方法参数
```

以上是使用反射的基础步骤，但如果要进一步学习反射，还需要对反射的常用API有更多的了解。

#### 关于Class类

- 要想解剖一个类,必须先要获取到该类的字节码文件对象（class）。通过Class类中的方法来获取class中的各种信息。
- <u>Class对象在类加载的时候创建，并存储在堆中，一个类只能有一个Class对象</u>
- 用于获取与类相关的各种信息，提供了获取类信息的相关方法

- 关于类加载的过程和机制会单独编写一篇文章来叙述（待完成）

![image-20200812093943295](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20200812094302.png)

## 二、反射常用API

在 JDK 中，反射相关的 API 可以分为下面几个方面：获取反射的 Class 对象、通过反射创建类对象、通过反射获取类属性方法及构造器。

#### 获取Class对象

第一种：使用 Class.forName 静态方法。

```java
Class clz = Class.forName("reflection.Apple"); // 类全路径名(String)
```

​	传入类型为字符串，适用于不知道要使用哪个类的情况

第二种：使用使用 .class方法

```java
Class clz = Apple.class; // 直接输入类名
```

​	类名前可以加上包名，默认本包，适用于知道具体要使用哪个类的情况

第三种：使用类对象的 getClass方法

```java
Apple apple = new Apple(); 
Class clz = apple.getClass(); // 使用实例对象获取Class对象
```

#### 通过Class获取类的信息

1、获取类的所有public变量

```java
Method[] method = clz.getMethods();
Field[] field = clz.getFields();
```

2、获取所有的构造方法

```java
Constructor[] constructors = clz.getConstructors();
```

3、获取所有成员变量包括private

```java
// 获取所有方法，包括private
Method[] declaredMethods = clz.getDeclaredMethods(); 
// 获取所有属性，包括private
Field[] declaredFields = clz.getDeclaredFields();
```

也可以传入变量来获取指定方法

#### 通过反射获取对象实例

创建反射类实例对象主要有两种方式，第一种是直接调用Class对象的newInstance() 方法，默认调用无参构造方法。第二种调用Constructor 对象的 newInstance() 方法，可选指定构造方法。

第一种：通过Class对象的newInstance()方法

```java
Class clz = Apple.class;
Apple apple = (Apple)clz.newInstance();
```

第二种：通过Constructor对象的newInstance()方法

```java
Class clz = Apple.class;
Constructor appleConstructor = clz.getConstructor();// 可传入参数调用指定构造
Apple appleObj = (Apple) appleConstructor.newInstance();
```

获取对象实例后，就可以操作对象，但是在使用反射的大多数情况下，我们无法知道反射的对象类型，因此不能直接调用对象的方法，要使用invoke来配合使用。

<u>问题：为什么newInstance后不直接调用对象实例的方法？</u>

因为不确定 反射出来的对象类型是什么，所以不知道有什么方法。如果明确知道对象的类型，那么当然可以强制转换后直接调用对象方法。

所以我们要使用method对象的invoke来调用对象方法：

```java
Method setKindMethod = clz.getMethod("setKind", String.class);
setKindMethod.invoke(appleObj, "red");// 对象实例，方法参数
```



在程序执行之前，程序不知道具体的类名和方法名，只能通过配置文件反射获取，Spring中就大量采用了反射机制。



参考：

[SpringBoot学习系列之一（反射）](https://www.cnblogs.com/YJzhiqianni/p/11324312.html)

[大白话说Java反射：入门、使用、原理](https://www.cnblogs.com/chanshuyi/p/head_first_of_reflection.html)

