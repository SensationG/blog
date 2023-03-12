---
layout: post
title: SpringCloud学习
date: 2021-03-21
Author: hhw
comments: true
toc: true
tags:  [java,Spring,微服务]
pinned: false
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

![image-20210320132342827](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210325175121.png)

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

### 1. 什么是Ribbon

> Spring Cloud Ribbon是基于Netflflix Ribbon实现的一套**客户端负载均衡的工具**。
>
> 主要功能是提供**客户端的**软件负载均衡算法（调用方即客户端本身具备负载均衡的功能）
>
> 如图：
>
> 1、在消费微服务中使用Ribbon实现负载均衡，**Ribbon先从EurekaServer中获取服务列表。**
> 2、Ribbon根据负载均衡的算法去调用微服务。

![image-20210325230250367](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210401152138.png)  

**1.1 负载均衡分类**

SpringCloud的负载均衡算法可以自定义

负载均衡简单分类：

- 集中式LB

  即在服务的消费方和提供方之间使用独立的LB设施

  如之前学习的<u>Nginx</u>，由该设施负责把访问请求通过某种策略转发至服务的提供方！

- 进程式LB

  将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地

  址中选出一个合适的服务器。

  <u>Ribbon就属于进程内LB</u>，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务

  提供方的地址！

  Ribbon的github地址 ： https://github.com/NetFlix/ribbon

### 2. Ribbon的工作

Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号

Ribbon在工作时分成两步

第一步先选择 EurekaServer，它优先选择在同一个区域内负载均衡较少的Server。

第二步在根据用户指定的策略，在从server去到的服务注册列表中选择一个地址。

其中Ribbon提供了多种策略，比如**轮询**（默认），随机和根据响应时间加权重,,,等等，可以通过java配置去修改策略，也可以自定义策略，Ribbon的核心组件包含了各种策略：

> RoundRobinRule【轮询】
>
> RandomRule【随机】
>
> AvailabilityFilterRule【会先过滤掉由于多次访问故障而处于断路器跳闸的服务，还有并发的连接
>
> 数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问】
>
> WeightedResponseTimeRule【根据平均响应时间计算所有服务的权重，响应时间越快服务权重越
>
> 大，被选中的概率越高，刚启动时如果统计信息不足，则使用RoundRobinRule策略，等待统计信
>
> 息足够，会切换到WeightedResponseTimeRule】
>
> RetryRule【先按照RoundRobinRule的策略获取服务，如果获取服务失败，则在指定时间内会进行
>
> 重试，获取可用的服务】
>
> BestAvailableRule【会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并
>
> 发量最小的服务】
>
> ZoneAvoidanceRule【默认规则，复合判断server所在区域的性能和server的可用性选择服务器】

（暂略实现步骤）

## 四、Feign

> feign是声明式的web service客户端，它让微服务之间的调用变得更简单了，类似controller调用
>
> service。
>
> Spring Cloud集成了Ribbon和Eureka，可在使用Feign时提供负载均衡的http客户端。
>
> 只需要创建一个接口，然后添加注解即可！
>
> 使用方法直接调接口即可

### 1. Feign解释

**1.1 Feign能干什么？**

- Feign旨在使编写Java Http客户端变得更容易

- 前面在使用Ribbon + RestTemplate时，利用RestTemplate对Http请求的封装处理，形成了一套模

  板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多

  处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，

  Feign在此基础上做了进一步封装，由他 来帮助我们定义和实现依赖服务接口的定义，**在Feign的实**

  **现下，我们只需要创建一个接口并使用注解的方式来配置它（类似于以前Dao接口上标注Mapper**

  **注解，现在是一个微服务接口上面标注一个Feign注解即可。）**即可完成对服务提供方的接口绑

  定，简化了使用Spring Cloud Ribbon时，自动封装服务调用客户端的开发量。

**1.2 Feign集成了Ribbon**

- 利用Ribbon维护了springcloud-Dept的服务列表信息，并且通过轮询实现了客户端的负载均衡，

  而与Ribbon不同的是，通过Feign只需要定义服务绑定接口且以声明式的方法，优雅而且简单的实

  现了服务调用。

### **2. Feign实现步骤**

**2.1 准备好注册中心，服务调用方，服务被调用方，被调用方对外暴露的接口**

<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210401152155.png" alt="image-20210328154347390" style="zoom:50%;" />

创建好模块后记得在父pom添加这个3个模块

```xml
<modules>
        <module>springcloud-api-feign</module>
        <module>eureka-client-6999</module>
        <module>erueka-client-6998</module>
        <module>eureka-server</module>
</modules>
```

**2.2 略过服务注册到注册中心的过程，且对外暴露接口无需启动类，首先在除了注册中心的三个包都引入Feign包，并在调用方引入对外暴露接口的包**

```xml
<!-- Feign支持 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-feign</artifactId>
  <version>1.4.6.RELEASE</version>
</dependency>
 <!-- 对外暴露接口的包，在服务调用方引入-->
 <dependency>
    <groupId>com.huang</groupId>
    <artifactId>springcloud-api-feign</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

**2.3 在调用方和被调用方启动类加上注解**

```java
// 这里一定要加入包扫描，否则会遇到AutoWired失败的问题，扫描对外暴露的接口
@EnableFeignClients(basePackages = {"com.huang"})
```

**2.4 编写调用方和被调用方接口、对外暴露接口包**

被调用方：

![image-20210328154956953](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210401152200.png)

对外暴露的API包：

使用FeignClient注解，value为被调用方服务名称，接口与被调用方接口URL相同

![image-20210328155045253](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210401152206.png)

调用方：

注入对外暴露的接口，直接调用该方法即可

![image-20210328155217647](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210401152210.png)

**2.5 通过请求调用方，可以直接获得被调用方返回的数据**

### 3. 小结

- Feign通过接口的方法调用Rest服务 ( 之前是Ribbon+RestTemplate )

-  <u>@FeignClient(value = serverName)即指定了serverName的服务名称，Feign会从注册中</u>
  <u>心获取服务列表，并通过负载均衡算法进行服务调用</u>

- 在接口方法 中使用注解@GetMapping("/cms/page/get/{id}")，指定调用的url，Feign将根据url进行远程调用。

- 通过Feign直接找到服务接口，由于在进行服务调用的时候融合了Ribbon技术，所以也支持负载均衡作

  用！

- feign其实不是做负载均衡的,负载均衡是ribbon的功能,feign只是集成了ribbon而已,但是负载均衡的功能还是feign内置的ribbon再做,而不是feign。

- feign的作用的替代RestTemplate,性能比较低，但是可以使代码可读性很强。

## 五、Hystrix

### 1. 分布式系统面临的问题

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免的失败！

![image-20210331174001600](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210401152219.png)

### 2. 服务雪崩

> 多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B 和微服务C又调用其他的微服务，这就是所谓的 “扇出”、如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的 “雪崩效应”。
>
> 对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒中内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障，这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

### 3. 什么是Hystrix  

> Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时，异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

“断路器” 本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），**向调用方返回一个服务预期的，可处理的备选响应（**FallBack），而不是长时间的等待或者抛出**调用方法无法处理的异常，这样就可以保证了服务调用方的线程不会被长时间**，不必要的占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

Hystrix提供的功能包括：

- 服务降级
- 服务熔断
- 服务限流
- 接近实时的监控
- .....

**官网资料**

https://github.com/Netflflix/Hystrix/wiki



### 4. 服务熔断

- 熔断机制是对应雪崩效应的一种微服务链路保护机制。

- 指某个服务超时或者异常，会引起服务熔断，但熔断是发生在服务内部的(并非是被调用的服务挂掉的情况，而是服务成功调用了，却卡在被调用的服务（可能是异常或超时）一直不返回的情况)，如果是被调用的服务挂掉了，需要用到服务降级

- 当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，**进而熔断该节点微服务的调**

  **用，快速返回 错误的响应信息**。当检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud

  框架里熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，**缺省**

  **是5秒内20次调用失败就会启动熔断机制。**

  熔断机制的注解是 `@HystrixCommand`

>  导入依赖=》在需要熔断的方法上面加入@HystriixCommand 指定熔断方法 =〉在启动类加上@EnableCircuitBreaker  或 @EnableHystrix （EnableHystrix包含了前者）断路器注解

**4.1 导入依赖**

- 在需要熔断期支持的服务中添加熔断包

```xml
<!--   Hystrix支持     -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
  <version>2.1.0.RELEASE</version>
</dependency>
```

**4.2 编写熔断备用方法**

- 这里以进入Controller为例子，服务抛出异常为例子

```java
  /**
     * Hystrix测试
     * @HystrixCommand 异常后进入指定的备用方法
     * @return
     */
@HystrixCommand(fallbackMethod = "hystrix")
@RequestMapping("/getUserHystrix")
public String getUserWithHystrix() {
  throw new RuntimeException();
}

public String hystrix() {
  return "进入Hystrix熔断器了！！！";
}
```

此时主方法抛出异常，会进入到指定的备用方法

**4.3 启动类加注解**

```java
// 加入熔断器
@EnableHystrix
```

启动类加上@EnableCircuitBreaker （会提示已经过时） 或 @EnableHystrix （EnableHystrix包含了前者）断路器注解

启动服务，请求进入该方法抛出异常后，会进入到hystrix方法中

#### 4.4 常见错误

 [org/springframework/boot/autoconfigure/web/ServerPropertiesAutoConfiguration.class] cannot be opened because it does not exist 错误

因为包引的不对 spring2.x.x>=2.0.0已经去掉了org/springframework/boot/autoconfigure/web/ServerPropertiesAutoConfiguration.class，如果导入的Jar包需要依赖这个就会报错

```xml
<!--断路器-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-hystrix</artifactId>
  <version>1.4.4.RELEASE</version>
</dependency>
```

替换为

```xml
 <!--断路器-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
  <version>2.1.0.RELEASE</version>
</dependency>
```

### 5. 服务降级

（测试结果失败，无法正常执行）

由客户端去实现，当服务关停的时候，客户端请求不到，这时就会出现对应的熔断提示，是由客户端去实现的。与服务熔断不同。

> 服务熔断：服务端～ 某个服务超市或者异常，引起熔断～，保险丝～
>
> 服务降级：客户端～ 从整理网站请求负载考虑～ 当某个服务熔断或者关闭之后，服务将不再被调用～，此时在客户端，我们准备一个FallbackFactory，返回一个默认的值（缺省值）

> **服务熔断：服务熔断是应对雪崩效应的一种微服务链路保护机制**。熔断的目的是当A服务模块中的某块程序出现故障后为了不影响其他客户端的请求而做出的及时回应。举个例子解释，生活中每家每户都在用电，小明家的电线因为故障导致了小明家停电了。而小李、小张家的电是正常使用的。电力公司没有因为小明家有故障线路而停掉其他人家的电，同时小明家没有使用有故障的电路的电。
>
> **服务降级：**例子：我们去银行排队办理业务，大部分的银行分为普通窗口、特殊窗口（VIP窗口，老年窗口）。某一天银行大厅排普通窗口的人巨多。这时特殊窗口贴出告示说某时刻之后再开放。那么这时特殊窗口的工作人员就可以空出来去帮其他窗口办理业务，提高办事效率，已达到解决普通窗口排队的人过的目的。这时即为降级，降级的目的是为了解决整体项目的压力，而牺牲掉某一服务模块而采取的措施。 => **把系统资源让给高优先级的服务**，例如双十一秒杀时候关闭退款、蚂蚁森林等

1. 触发原因不太一样，服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑；
2. 管理目标的层次不太一样，熔断其实是一个框架级的处理，每个微服务都需要（无层级之分），而降级一般需要对业务有层级之分（比如降级一般是从最外围服务开始）
3. 实现方式不太一样

![image-20210330224544898](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210401152230.png)

**5.1 配置服务降级**

在编写的feign接口的包中，新增一个类，要求实现 `FallBackFactory接口`，且包含匿名内部类：实现feign所在的service

```java
@Component // 需要加入Spring
public class FeignTestFallBackFactory implements FallbackFactory {

  @Override
  public Object create(Throwable cause) {
    return new FeignTestInterface() {

      @Override
      public String getUserInfo() {
        // 在这里配置 如果服务访问不到（服务关停） 需要返回什么
        return "服务关停啦！服务降级";
      }

      @Override
      public String getUserHystrix() {
        return "服务关停啦！服务降级";
      }
    };
    
  }
}
```

**5.2 在Feign所在的类中，新增注解**

在@FeignClient中添加fallbackFactory属性，值为5.1刚才新建的类

```java
@FeignClient(value = "EUREKA-CLIENT-6998", fallbackFactory = FeignTestFallBackFactory.class)
```

**5.3 客户端配置**

在客户端的applicaiton.yml中配置

```yml
# 开启服务降级
feign:
  circuitbreaker:
    enabled: true
```

当服务端关停时，客户端因反问不到而触发服务降级，从而返回上面配置的内容：服务关停啦，服务降级



### 6. Dashboard流监控

【略】

## 六、Zuul

### 1. 什么是Zuul

- Zuul包含了对请求的路由和过滤两个最主要的功能：

  其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础，而过滤器

  功能则负责对请求的处理过程进行干预，是实现请求校验，服务聚合等功能的基础。Zuul和Eureka进行

  整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后

  的访问微服务都是通过Zuul跳转后获得。

- 注意：Zuul服务最终还是会注册进Eureka

- 提供：代理 + 路由 + 过滤 三大功能！
- 与Nginx区别：
  - 相同点
    - nginx可以进行负载均衡，请求转发，但更多用于负载均衡
    - zuul可以进行请求转发，且集成ribbon也可以进行负载均衡，但更多用于请求转发
  - 不同点
    - nginx c语言，zuul java语言
    - nginx功能更加强大，因为Nginx整合一些脚本语言（Nginx+lua）

### 2. Zuul的作用

- 路由（请求转发）
- 过滤

官网文档：https://github.com/Netflflix/zuul

### 3. 实现步骤

经测试无法成功，怀疑是引入的包版本不对

【略】







