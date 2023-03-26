---
layout: post
title: Spring_event发布订阅模式
date: 2023-03-26
Author: hhw
comments: true
toc: true
tags:  [java,spring]
---

## 1、前言

> 在项目里，经常会有一些主线业务之外的其它业务，比如，下单之后，发送通知、监控埋点、记录日志……这些非核心业务，如果全部一梭子写下去，有两个问题，一个是业务耦合，一个是串行耗时。
>
> ![img](https://img-blog.csdnimg.cn/img_convert/dccb2846c06e4533ea88399aac335415.webp?x-oss-process=image/format,png)

>使用发布订阅模式来解决这些问题：把这些操作抽象成观察者模式，也就是发布/订阅模式（这里就不讨论观察者模式和发布/订阅模式的不同），而且一般会采用多线程的方式来异步执行这些观察者方法。
>
>![img](https://img-blog.csdnimg.cn/img_convert/2dc8ef28a999164690dbd8aa673b876f.webp?x-oss-process=image/format,png)

## 2、Spring Event实现发布/订阅模式

### 步1 定义事件(监听接口)

> 1、该事件接口继承ApplicationEvent，并重写构造函数。ApplicationEvent是Spring提供的所有应用程序事件扩展类。
>
> 2、<font color="blue">消息发布者通过调用此接口发布消息</font>

```java
import com.huanghwh.demo.event.entity.OrderMessage;
import org.springframework.context.ApplicationEvent;

/**
 * @Author: huanghwh
 * @Date: 2023/03/26 下午8:46
 * @Description: 订单事件监听事件(接口)：继承ApplicationEvent，并重写构造函数
 */
public class OrderEvent extends ApplicationEvent {
    
    public OrderEvent(OrderMessage orderMessage) {
        super(orderMessage);
    }
}
```

### 步2 消息传输实体

> 1、用于过程中传输信息的专用实体

```java
/**
 * @Author: huanghwh
 * @Date: 2023/03/26 下午8:47
 * @Description: 订单消息实体
 */
 @Data
public class OrderMessage implements Serializable {
    
    private String orderNo;
    
    private String orderName;

    public OrderMessage(String orderNo, String orderName) {
        this.orderNo = orderNo;
        this.orderName = orderName;
    }
}

```

### 步3 定义监听(消费)者

> 1、实际<font color="blue">消费者</font>，会消费指定接口中的消息
>
> 2、有2种方式：可使用注解@EventListener(OrderEvent.class)或实现ApplicationListener，二选一
>
> 3、这里定义了2种监听者为例
>
> 4、异步执行：@Async("myThreadExecutor") ，myThreadExecutor是自定义线程池

```java
import cn.hutool.log.StaticLog;
import com.huanghwh.demo.event.entity.OrderMessage;
import org.springframework.context.ApplicationListener;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

/**
 * @Author: huanghwh
 * @Date: 2023/03/26 下午8:45
 * @Description: 订单消费者1：日志记录
 */
@Service
public class OrderLogListener implements ApplicationListener<OrderEvent> {
    
    //@EventListener(OrderEvent.class) //可使用注解或实现ApplicationListener，二选一
    @Async("myThreadExecutor") // 异步执行
    @Override
    public void onApplicationEvent(OrderEvent orderEvent) {
        OrderMessage source = (OrderMessage) orderEvent.getSource();
        StaticLog.info("异步消费者1:日志记录，订单号{}", source.getOrderNo());
    }
    
}
```

```java
import cn.hutool.log.StaticLog;
import com.huanghwh.demo.event.entity.OrderMessage;
import org.springframework.context.ApplicationListener;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

/**
 * @Author: huanghwh
 * @Date: 2023/03/26 下午8:45
 * @Description: 订单消费者2：消息通知
 */
@Service
public class OrderMessageListener implements ApplicationListener<OrderEvent> {

    //@EventListener(OrderEvent.class) //可使用注解或实现ApplicationListener，二选一
    @Async("myThreadExecutor") // 异步执行
    @Override
    public void onApplicationEvent(OrderEvent orderEvent) {
        OrderMessage source = (OrderMessage) orderEvent.getSource();
        StaticLog.info("异步消费者2:消息通知，订单号{}", source.getOrderNo());
    }
    
}
```

### 步4 发布消息

> 1、<font color="blue">使用Spring 提供的`ApplicationEventPublisher`来发布自定义事件。</font>

```java
import cn.hutool.log.StaticLog;
import com.huanghwh.demo.event.entity.OrderMessage;
import com.huanghwh.demo.event.listener.OrderEvent;
import com.huanghwh.demo.event.publisher.IOrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;

/**
 * @Author: huanghwh
 * @Date: 2023/03/26 下午9:50
 * @Description: 消息发布demo
 */
@Service
public class OrderServiceImpl implements IOrderService {
    
    /* 使用Spring 提供的ApplicationEventPublisher来发布自定义事件 */
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;
    
    @Override
    public void saveOrder() {
        StaticLog.info("生产者：准备生成订单...");
        // ..生成订单(略)
        String orderNum = "001";
        String orderName = "眼镜";
        // 订单消息类
        StaticLog.info("准备发布订单号：{}", orderNum);
        OrderMessage orderMessage = new OrderMessage(orderNum, orderName);
        // 发布消息 入参: 订单事件监听接口(消息传输实体类)
        applicationEventPublisher.publishEvent(new OrderEvent(orderMessage));
        StaticLog.info("发布完成：{}", orderNum);
    }
}
```

## 3、测试

### 1.引入包

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

### 2.测试方法编写

> 1、注意：测试类需要和springboot启动类处于同一层目录下

```java
/**
 * @Author: huanghwh
 * @Date: 2023/03/26 下午9:58
 * @Description:
 */
@SpringBootTest
public class OrderTest {

    @Autowired
    private IOrderService orderService;

    /**
     * 测试发布订单消息
     */
    @Test
    public void testPublish() {
        orderService.saveOrder();
    }
}
```

输出：

```log
2023-03-26 22:01:29.759  INFO 57428 --- [           main] c.h.d.e.publisher.impl.OrderServiceImpl  : 生产者：准备生成订单...
2023-03-26 22:01:29.765  INFO 57428 --- [           main] c.h.d.e.publisher.impl.OrderServiceImpl  : 准备发布订单号：001
2023-03-26 22:01:29.771  INFO 57428 --- [thread-execute1] c.h.d.event.listener.OrderLogListener    : 异步消费者1:日志记录，订单号001
2023-03-26 22:01:29.771  INFO 57428 --- [           main] c.h.d.e.publisher.impl.OrderServiceImpl  : 发布完成：001
2023-03-26 22:01:29.771  INFO 57428 --- [thread-execute2] c.h.d.e.listener.OrderMessageListener    : 异步消费者2:消息通知，订单号001
```

参考：https://blog.csdn.net/BASK2312/article/details/127961590
