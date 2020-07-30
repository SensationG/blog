---
layout: post
title: Java异常抛出后还会执行吗？
date: 2020-02-22
Author: hhw
comments: true
tags:  [java]
---

## 问题：
写业务代码的时候发现了一个问题，处理异常的时候需要使用`try catch`捕获还是使用`throw` 抛出，这二者在代码执行流程上有何区别？`try catch`之后的代码还会执行吗？
## 测试：
- 案例一

    ```java
    public static int test() {
        throw new RuntimeException();
        return 1; //编译错误，提示：无法访问的语句
    }
    ```
    总结：`throw`后的语句不能被执行。
- 案例二

    ```java
    public static int test() {
        try {
            throw new RuntimeException();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return 1; //正常执行
    }
    ```
    结论：使用`try catch`捕获异常后（前提是抛出的异常符合catch捕获的异常类型或其子类，才能成功在carch捕获），后续的代码可以正常执行<br>
    ***变形*** ：catch中return了，那么后续的return还会执行吗？
     ```java
    public static int test() {
        try {
            throw new RuntimeException();
        } catch (Exception e) {
            e.printStackTrace();
            return 0; //捕获到异常后，正常执行
        }
        return 1; //catch中retun了，这里不会执行
    }
     ```
    结论：如果我们不想后续的程序终止，可以使用捕获的方式；并且通过改变return的位置来控制返回的结果。
- 案例三

    ```java
    public static int test() {
       if (true) {
           throw new RuntimeException();
       }
       return 1; //上面抛出异常，这一步不会执行
    }
    ```
    结论：我们可以通过这种案例，手动抛出异常，配合`@transaction`注解进行事务处理。
