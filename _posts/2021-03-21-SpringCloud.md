---
layout: post
title: SpringCloud学习
date: 2020-03-21
Author: hhw
comments: true
toc: true
tags:  [java,Spring,微服务]
pinned: true
---

## 前言

**集群**：将一个整体复制多份，每份都是相同的

**分布式**：将一个项目拆成多份，例如一个购物系统，将支付、物流环节拆出来单独形成一个服务，分开成不同的服务器去完成。只有当单个节点的处理能力无法满足日益增长的计算的时候，且硬件的提升高昂得不偿失的时候，我们才需要考虑分布式。而由于分布式多节点，通过网络通信的拓扑结构，会引入单机系统没有的问题，例如协议、机制等等，需要考虑更多的问题

## 一、什么是微服务/Cloud

### 1. 什么是微服务

> **微服务是一种架构模式，或者说是一种架构风格，它提倡将单一的应用程序划分为一组小的服务，每个服务允许在其独立的进程内，服务之间相互协调、配置，为客户提供最终的价值**
>
> 服务之间采用轻量级的通信机制互相沟通，每个服务都围绕着具体的业务进行构建，并且能够被独立地部署到生产环境中

**可能有的人觉得官方的话太过生涩，我们从技术维度来理解下：**

> **微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合，每一个微**

> **服务提供单个业务功能的服务，一个服务做一件事情，从技术角度看就是一种小而独立的处理过程，类似进程的概念，能够自行单独启动或销毁，拥有自己独立的数据库。**



微服务优缺点：

优点：

- 每个服务足够内聚，足够小，代码容易理解，这样能聚焦一个指定的业务功能或业务需求；

- 开发简单，开发效率提高，一个服务可能就是专一的只干一件事；

- 微服务能够被小团队单独开发，这个小团队是2~5人的开发人员组成；

- 微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或部署阶段都是独立的。

- 微服务能使用不同的语言开发。

- 易于和第三方集成，微服务允许容易且灵活的方式集成自动部署，通过持续集成工具，如jenkins，

- Hudson，bamboo

- 微服务易于被一个开发人员理解，修改和维护，这样小团队能够更关注自己的工作成果。无需通过

- 合作才能体现价值。

- 微服务允许你利用融合最新技术。

- **微服务只是业务逻辑的代码，不会和** **HTML** **，** **CSS** **或其他界面混合**

- **每个微服务都有自己的存储能力，可以有自己的数据库，也可以有统一数据库**

缺点：

- 开发人员要处理分布式系统的复杂性

- 多服务运维难度，随着服务的增加，运维的压力也在增大

- 系统部署依赖

- 服务间通信成本

- 数据一致性

- 系统集成测试

- 性能监控.....



微服务架构4个核心问题？

1. 服务很多，客户端如何进行访问？
2. 这么多服务，服务之间如何进行通信？
3. 这么多服务，如何治理？
4. 服务挂了怎么办？

解决方案：

​	Spring Cloud  它是一个生态，参杂了很多东西包括 基于SpringBoot

​	下面是三种微服务的技术选型：

​	<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204044.png" alt="image-20210314215543644" style="zoom:47%;" />

cloud万变不离其宗：

1. API
2. HTTP，RPC
3. 注册和发现
4. 熔断机制

### 2. SpringCloud

本篇文章学习SpringCloud Netflix

**SpringCloud**

- 基于HTTP restful风格进行通信 （区别于RPC通信）
- cloud用来解决分布式带来的问题，如通信、访问、熔断



![image-20210314220053836](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204057.png)

### 3. SpringBoot&SpringCloud区别

- SpringBoot专注于快速方便的开发单个个体微服务。

- SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，为各个微服务之间提供：配置管理，服务发现，断路器，路由，微代理，事件总线，全局锁，决策竞选，分布式会话等等集成服务。

- SpringBoot可以离开SpringCloud独立使用，开发项目，但是SpringCloud离不开SpringBoot，属于依

  赖关系

- **SpringBoot专注于快速、方便的开发单个个体微服务，SpringCloud关注全局的服务治理框架**

### 4. Dubbo和SpringCloud

Dubbo和SpringCloud的服务区别

![image-20210316224314015](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321205452.png)

springcloud提供了完善的生态

### 5. 官方参考文档

参考：

- https://springcloud.cc/spring-cloud-netflflix.html

- 中文API文档：https://springcloud.cc/spring-cloud-dalston.html

- SpringCloud中国社区 http://springcloud.cn/

- SpringCloud中文网 https://springcloud.cc



## 二、Eureka

### 1. 基本介绍

**服务的注册与发现**

- Eureka是Netflflix的一个子模块，也是核心模块之一。<u>Eureka是一个基于REST的服务</u>，<u>用于定位服务</u>，以实现云端中间层服务发现和故障转移，服务注册与发现对于微服务来说是非常重要的，有了服务发现与注册，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了，<u>功能类似于Dubbo的注册中心</u>，比如Zookeeper； 

- Netflix在设计Eureka时，遵循的就是CAP原则

  ```
  CAP原则又称CAP定理，指的是在一个分布式系统中 一致性（Consistency） 可用性（Availability） 分区容错性（Partition tolerance） CAP 原则指的是，这三个要素最多只能同时实现两点，不可能三者兼顾。
  ```



**基本架构**

- SpringCloud 封装了NetFlix公司开发的Eureka模块来实现服务注册和发现
- Eureka采用了C-S的架构设计，EurekaServer 作为服务注册功能的服务器，他是服务注册的中心（服务端）
- 系统中的其他微服务使用Eureka的客户端（客户端）连接到EurekaServer（服务端）并维持心跳链接。这样系统的维护人员可以通过EurekaServer来监控系统中各个微服务是否正常运行，SpringCloud的一些其他模块（比如Zuul）就可以通过EurekaServer来发现系统中的其他微服务，并执行相关的逻辑；

![image-20210316222159349](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204112.png)

（基于REST进行远程调用）

- 与Dubbo对比

  ![image-20210316224851203](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204117.png)

- **Eureka主要包含两个组件：**

  - Eureka Server
  - Eureka Client

  **Eureka Server** 提供服务注册服务，各个节点启动后，会在EurekaServer中进行注册，这样Eureka

  Server中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看

  到。

  **Eureka Client**是一个Java客户端，用于简化EurekaServer的交互，客户端同时也具备一个内置的，使用轮询负载算法的负载均衡器。在应用启动后，将会向EurekaServer发送**心跳**（默认周期为30秒）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除掉（默认周期为90秒）

- 三大角色

  - Eureka Server：提供服务的注册于发现。
  - Service Provider：将自身服务注册到Eureka中，从而使消费方能够找到。
  - Service Consumer：服务消费方从Eureka中获取注册服务列表，从而找到消费服务。



### 2. 搭建总项目

> 搭建父级Maven工程 参考[网站](https://blog.csdn.net/weixin_43758283/article/details/84664556)

**2.1 新建Project**

![image-20210317230212824](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204120.png)



**2.2 type选择 Maven POM，父级只需要一个POM，其余都不需要**

![image-20210317230446525](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204124.png)

**2.3 选择cloud核心组件**

![image-20210317230512546](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204137.png)

**2.4 确定项目名称 点击完成**

![image-20210317230543544](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204140.png)

**2.5 修改pom.xml**

注意 在使用spring创建项目时，已经选择了版本信息，创建的版本信息已经匹配好了，不要修改 `spring-boot-starter-parent`以及`spring-cloud-starter`的版本号，否则启动会报错；

到这步创建完毕，如果报错，注意先clean 再package，然后再刷新

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.huang</groupId>
    <artifactId>sc-parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>sc-parent</name>
    <description>Demo project for Spring Boot</description>

    <!--打包方式  pom-->
    <packaging>pom</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

  	<!-- 上面的spring-boot-starter和下面的spring-cloud.version版本信息不能修改，否则报错 -->
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>2020.0.1</spring-cloud.version>
        <junit.version>4.12</junit.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter</artifactId>
        </dependency>

       <!-- 以下由于maven导包报错 将报错的包导入可以解决报错问题 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>

        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.6</version>
        </dependency>

        <dependency>
            <groupId>org.hdrhistogram</groupId>
            <artifactId>HdrHistogram</artifactId>
            <version>2.1.10</version>
        </dependency>

        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.10</version>
        </dependency>

    </dependencies>
  
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```



### 3. 编写Eureka服务端

**3.1 新建子模块**

![image-20210318092919754](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204145.png)

**3.2 选择maven project**

![image-20210318093006242](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204148.png)

**3.3 选择 Eureka Server**

![image-20210318093101775](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204151.png)

**3.4 选择项目目录 要注意一定要在父项目下，否则可能会覆盖父pom**

![image-20210318093143298](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204154.png)

**3.5 修改pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <name>eureka-server</name>
    <description>Demo project for Spring Boot</description>

    <!-- 继承父pom -->
    <parent>
        <groupId>com.huang</groupId>
        <artifactId>sc-parent</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>2020.0.1</spring-cloud.version>
    </properties>

    <dependencies>

        <!-- eureka服务端包 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

    </dependencies>

</project>

```

**3.6 在主启动类添加注解 @EnableEurekaServer**

 ```java
/**
 * 启用Eureka服务端
 */
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
 ```

**3.7 在全局配置文件进行配置 appplication.yml**

```yml
spring:
  application:
    name: springcloud-config-eureka-7001
server:
  port: 7001

# eureka服务端配置
eureka:
  instance:
    # 注册中心主机名
    hostname: localhost
  client:
    # 是否将自己注册到eureka服务器中，本身是服务器，无需注册
    register-with-eureka: false
    # 是否检索服务 false由于这里是注册中心服务端，负责维护服务实例，无需检索服务
    fetch-registry: false
    # 指定注册中心的位置
    service-url:
      # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个defaultZone地址
      # Eureka控制台示例访问路径 http://localhost:7001/ 即可 不需要+/eureka/
      defaultZone:  http://${eureka.instance.hostname}:${server.port}/eureka/
```

**3.7 启动**

访问 本机IP + 端口号即可 ；例如：http://localhost:7001/



### 4. 编写Eureka客户端

**4.1 创建客户端包**

与之前创建服务端类似，创建时选择：

![image-20210318135106668](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204159.png)

**4.2 创建完成后，编写pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.huang</groupId>
    <artifactId>eureka-client-7000</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <name>eureka-client-7000</name>
    <description>Demo project for eureka-client</description>

    <!-- 继承父pom -->
    <parent>
        <groupId>com.huang</groupId>
        <artifactId>sc-parent</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>2020.0.1</spring-cloud.version>
    </properties>

    <dependencies>
        <!-- erueka客户端包支持 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 需要这个依赖才能正常启动 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**4.3 编写application.yml**

```yml
server:
  port: 7000

eureka:
  client:
    service-url:
      # 注册中心地址(eureka服务端地址)
      defaultZone: http://localhost:7001/eureka/

spring:
  application:
    # 本服务在注册中心的服务名称
    name: eureka-client-7000
```

**4.4 在主启动类加上 @EnableEurekaClient**

```java
/**
 * @EnableEurekaClient eureka客户端
 */
@EnableEurekaClient
@SpringBootApplication
public class EurekaClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }

}

```

启动后，可以在服务端看到成功注册的客户端

![image-20210318135405109](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204204.png)

可以通过修改配置文件，修改eureka上该服务的默认描述信息（status）

```yml
eureka:
  instance:
    # 修改eureka上的默认描述信息
    instance-id: springcloud-client-7000
```

### 5. 自我保护机制

![image-20210320132342827](SpringCloud笔记.assets/image-20210320132342827.png)

出现红色情况的时候，是激活了Eureka的自我保护机制（之前出现的这些红色情况，没出现的，修改一个服务名，故意制造错误！）

> 一句话总结：某时刻某一个微服务不可以用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存！

**5.1 自我保护背景**

首先对Eureka注册中心需要了解的是Eureka各个节点都是平等的，没有ZK中角色的概念， 即使N-1个节点挂掉也不会影响其他节点的正常运行。

默认情况下，**如果Eureka Server在一定时间内（默认90秒）没有接收到某个微服务实例的心跳，Eureka Server将会移除该实例。**但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通信，而微服务本身是正常运行的，此时不应该移除这个微服务，所以引入了自我保护机制。

**5.2 自我保护机制**

官方对于自我保护机制的定义：

> 自我保护模式正是一种针对网络异常波动的安全保护措施，使用自我保护模式能使Eureka集群更加的健壮、稳定的运行。

自我保护机制的工作机制是：**如果在15分钟内超过85%的客户端节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，Eureka Server自动进入自我保护机制**，此时会出现以下几种情况：

1. Eureka Server不再从注册列表中移除因为长时间没收到心跳而应该过期的服务。
2. Eureka Server仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上，保证当前节点依然可用。
3. 当网络稳定时，当前Eureka Server新的注册信息会被同步到其它节点中。

因此Eureka Server可以很好的应对因网络故障导致部分节点失联的情况，而不会像ZK那样如果有一半不可用的情况会导致整个集群不可用而变成瘫痪。

在自我保护模式中，EurekaServer会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心

跳数重新恢复到阈值以上时，该EurekaServer节点就会自动退出自我保护模式。它的设计哲学就是宁可

保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话：好死不如赖活着。

**5.3 自我保护开关**

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务

（健康的微服务和不健康的微服务都会保留），也不盲目注销任何健康的微服务。使用自我保护模式，

可以让Eureka集群更加的健壮和稳定。

在SpringCloud中，可以使用`eureka.server.enable-self-preservation = false` 禁用自我保护

模式 【不推荐关闭自我保护机制】



### 6. Eureka集群

> 防止因为一台eureka宕机后 所有的服务都宕机无法相互访问

**6.1 修改hosts**

由于只有一台本机，为了实现模拟集群的效果，修改hosts来实现 不同域名都映射到localhost

```
127.0.0.1 eurekaserver7001.com
127.0.0.1 eurekaserver7002.com
127.0.0.1 eurekaserver7003.com
```

**6.2 建立其他2个eureka server工程**

与之前搭建好的eureka server组成3台集群，搭建方式参照上面的eureka服务端

搭建完成后的pom可以参照之前的服务端pom文件，application.xml配置如下

集群服务1: Eureka7001

```yml
spring:
  application:
    name: springcloud-config-eureka-7001
server:
  port: 7001

# eureka服务端配置
eureka:
  instance:
    # 注册中心主机名
    hostname: eurekaserver7001
  client:
    # 是否将自己注册到eureka服务器中，本身是服务器，无需注册
    register-with-eureka: false
    # 是否检索服务 false由于这里是注册中心服务端，负责维护服务实例，无需检索服务
    fetch-registry: false
    # 指定注册中心的位置
    service-url:
      # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个defaultZone地址
      # Eureka控制台示例访问路径 http://localhost:7001/ 即可 不需要+/eureka/
      # 单机配置 需要自己注册自己 defaultZone:  http://${eureka.instance.hostname}:${server.port}/eureka/
      # 集群配置 使用集群，不需要自己注册自己，以下为其他两个eureka服务注册中心地址
      defaultZone: http://eurekaserver7002.com:7002/eureka/,http://eurekaserver7003.com:7003/eureka/
```

集群服务2: Eureka7002

```yml
spring:
  application:
    name: springcloud-config-eureka-7002
server:
  port: 7002

# eureka服务端配置
eureka:
  instance:
    # 注册中心主机名
    hostname: eureka7002.com
  client:
    # 是否将自己注册到eureka服务器中，本身是服务器，无需注册
    register-with-eureka: false
    # 是否检索服务 false由于这里是注册中心服务端，负责维护服务实例，无需检索服务
    fetch-registry: false
    # 指定注册中心的位置
    service-url:
      # 集群配置 使用逗号隔开即可
      defaultZone:  http://eurekaserver7001.com:7001/eureka/,http://eurekaserver7003.com:7003/eureka/
```

集群服务3: Eureka7003

```yml
spring:
  application:
    name: springcloud-config-eureka-7003
server:
  port: 7003

# eureka服务端配置
eureka:
  instance:
    # 注册中心主机名
    hostname: eurekaserver7003
  client:
    # 是否将自己注册到eureka服务器中，本身是服务器，无需注册
    register-with-eureka: false
    # 是否检索服务 false由于这里是注册中心服务端，负责维护服务实例，无需检索服务
    fetch-registry: false
    # 指定注册中心的位置
    service-url:
      # 集群配置
      defaultZone:   http://eurekaserver7001.com:7001/eureka/,http://eurekaserver7002.com:7002/eureka/
```

**6.3 启动3个Server**

访问任意erueka服务端控制台，可以看到另外两个Server

```
http://eurekaserver7001.com:7001/
http://eurekaserver7002.com:7002/
http://eurekaserver7003.com:7003/
```

<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204213.png" alt="image-20210319105954294" style="zoom:50%;" />

将client客户端微服务发布到1台eureka集群配置中，发现在集群中的其余注册中心也可以看到：

![image-20210319110049026](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210321204218.png)

也可以选择让client在3个集群eureka中都注册：

```yml
# eureka 配置
eureka:
  client:
    service-url:
      defaultZone: http://eurekaserver7001.com:7001/eureka/,http://eurekaserver7002.com:7002/eureka/,http://eurekaserver7003.com:7003/eureka/
```



### 7. CAP原则

**7.1 什么是CAP**

C（Consistency） 强一致性

A（Availability）可用性

P（Partition tolerance）分区容错性

CAP的三进二：CA、AP、CP

**7.2 CAP的核心理论**

- 一个分布式系统不可能很好地满足一致性，可用性和分区容错性这三个需求
- 根据CAP原理，将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类：
  - CA：单点集群，满足一致性，可用性的系统，通常可扩展性较差
  - CP：满足一致性，分区容错性的系统，通常性能不是特别高
  - AP：满足可用性，分区容错性的系统，通常可能对一致性要求低一些

**7.3 Eureka 和 ZooKeeper**

著名的CAP理论指出，一个分布式系统不可能同时满足C（一致性）、A（可用性）、P（容错性）。

<u>由于分区容错性P在分布式系统中是必须要保证的，因此我们只能在A和C之间进行权衡。</u>

Zookeeper保证的是CP；

Eureka保证的是AP；

**Zookeeper**保证的是**CP**

当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服

务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。但是zk会出现这样一种

情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在

于，选举leader的时间太长，30~120s，且选举期间整个zk集群都是不可用的，这就导致在选举期间注

册服务瘫痪。在云部署的环境下，因为网络问题使得zk集群失去master节点是较大概率会发生的事件，

虽然服务最终能够恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

**Eureka**保证的是**AP**

Eureka看明白了这一点，因此在设计时就优先保证可用性。**Eureka**各个节点都是平等的，几个节点挂

掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个

Eureka注册时，如果发现连接失败，则会自动切换至其他节点，只要有一台Eureka还在，就能保住注册

服务的可用性，只不过查到的信息可能不是最新的，除此之外，Eureka还有一种自我保护机制，如果在

15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此

时会出现以下几种情况：

1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务

2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上（即保证当前节点依

然可用）

3. 当网络稳定时，当前实例新的注册信息会被同步到其他节点中

**因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整**

**个注册服务瘫痪**



## 三、Ribbon









## 四、Feign

## 五、Hystrix

## 六、Zuul

## 七、Config

