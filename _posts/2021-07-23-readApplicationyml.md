---
layout: post
title: Spring Boot 静态变量@value取不到值
date: 2021-07-23
Author: hhw
comments: true
tags:  [Spring Boot,Spring]
---

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









