---
layout: post
title: Spring Boot @Value 注解使用 (解决静态变量@value取不到值)
date: 2021-07-23
Author: hhw
comments: true
toc: true
tags:  [Spring Boot,Spring]
---

## 1. 解决静态变量@value取不到值

**1.** 平时用的时候，直接在变量头上加上@Value就能取到对应配置文件中的值

```java
@Value("${ephone.notPushProjectName}")
private String notPushProjectName;
```

**2.** 但是当savePath被static修饰了之后，就赋不了值

```java
@Value("${ephone.notPushProjectName}")
private static String notPushProjectName;
```

这是因为Spring Boot 不支持/不允许把值注入到静态变量中，但是也给出了解决的方案

**3.** 把`@Value("${ephone.notPushProjectName}")`放到静态变量的set方法上面即可，需要注意的是set方法要去掉static，还有就是当前类要交给spring来管理（使用`@Component`注解）

```java
@Component // 1.记得加注解
public class PropConstant {

　　private static String notPushProjectName;
　　// 2. 使用set方法设置值 记得去掉static 
　　@Value("${ephone.notPushProjectName}")
    public void setNotPushProjectName(String notPushProjectName){
        SendEPhoneUtil.notPushProjectName = notPushProjectName;
    }
}
```



## 2. 使用@Value解析List

以使用 `.yml` 文件为例，我们只需要在配置文件中，跟配置数组一样去配置：

```yml
test:
  list: aaa,bbb,ccc
```

在调用时，借助 `EL` 表达式的 `split()` 函数进行切分即可。

```java
@Value("#{'${test.list}'.split(',')}")
private List<String> testList;
```

同样，为它加上默认值，避免不配置这个 key 时候程序报错：

```java
@Value("#{'${test.list:}'.split(',')}")
private List<String> testList;
```

但是这样有个问题，当不配置该 key 值，默认值会为空串，它的 length = 1（不同于数组，length = 0），这样解析出来 list 的元素个数就不是空了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1QxwhpDy7ia3fQR4Umz06MJsMic2cZr4sPTB6ibI9HhGkic3PpibzOuUR8enAlYlQYRZdbicJrqpSrjKf4AKjIbDElLw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这个问题比较严重，因为它会导致代码中的判空逻辑执行错误。这个问题也是可以解决的，在 `split()` 之前判断下是否为空即可。

```java
@Value("#{'${test.list:}'.empty ? null : '${test.list:}'.split(',')}")
private List<String> testList;
```

如上所示，即为最终的版本，它具有数组方式的全部优点，且更容易在业务代码中去应用。





