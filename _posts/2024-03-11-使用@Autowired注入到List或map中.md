---
layout: post
title: 使用@Autowired注入到List或map中
date: 2024-03-11
Author: hhw
comments: true
toc: true
tags:  [java]
---

# 一、将bean注入到List或Map

> 在SpringBoot开发中，当一个接口A有多个实现类时，spring会很智能的将bean注入到List<A>或Map<String, A>变量中。我们可以使用它来实现策略+工厂模式，对业务代码进行解耦。

## 步骤1：定义一个接口

```java
public interface IAnimal {
    
    /**
     * 吃
     */
    void eat();
}
```

定义一个动物接口，他们都有吃饭的行为。

## 步骤2：定义多个实现

```java
@Slf4j
@Service
public class Cat implements IAnimal {
    
    @Override
    public void eat() {
        log.info("猫吃饭");
    }
}
```

```java
@Slf4j
@Service
public class Dog implements IAnimal {
    
    @Override
    public void eat() {
        log.info("狗吃饭");
    }
}
```

定义猫、狗的实现类，具体实现“怎么吃”。

## 步骤3：使用@Autowired注入

### 注入List

```java
/*使用List注入多个实现类*/
@Autowired
List<IAnimal> animalList;

@GetMapping(value = "/getAnimal")
public void getAnimal() {
    log.info("打印animalList");
    for (IAnimal animal : animalList) {
        animal.eat();
    }
}
```

List中包含具体的实现类实例。

### 注入Map

```java
/*使用map注入多个实现类 key必须为String类型，即bean的名称，而value为IPerson类型的对象实例。*/
@Autowired
Map<String, IAnimal> animalMap;

@GetMapping(value = "/getAnimal")
public void getAnimal() {
    log.info("打印animalMap");
    for (Map.Entry<String, IAnimal> entry : animalMap.entrySet()) {
        log.info("IAnimal:" + entry.getKey() + ", " + entry.getValue());
    }

}

// 策略模式+简单工厂
@GetMapping(value = "/getAnimalByType")
public void getAnimalByType(String type) {
    IAnimal animal = animalMap.get(type);
    animal.eat();
}
```

key必须为String类型，即bean的名称，而value为IPerson类型的对象实例。我们可以通过传入beanName从而获取到对应的实例，实现了工厂模式；而们定义统一的动物接口，动物们自己去实现各自的吃的方法，实现了策略模式。（类的单一职责、解耦）







