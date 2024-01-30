---
layout: post
title: ArrayList快速初始化方法
date: 2023-07-31
Author: hhw
comments: true
toc: true
tags:  [java]

---

> 当我们要快速初始化包含多个元素的ArrayList时，该如何进行？本文包含了4种初始化方法

# 1、Arrays.asList

```java
ArrayList<Type> obj = new ArrayList<Type>(Arrays.asList(Object o1, Object o2, Object o3, ....so on));
```

# 2、生成匿名内部内进行初始化

```java
ArrayList<T> obj = new ArrayList<T>() {{
    add(Object o1);
    add(Object o2);
    ...
    ...
}};
```

# 3、常规方式

```java
ArrayList<T> obj = new ArrayList<T>();
obj.add("o1");
obj.add("o2");
...
...
```

或

```java
ArrayList<T> obj = new ArrayList<T>();
List list = Arrays.asList("o1","o2",...);
obj.addAll(list);
```

# 4、Collections.ncopies

```java
ArrayList<T> obj = new ArrayList<T>(Collections.nCopies(count,element));//把element复制count次填入ArrayList中
```









