---
layout: post
title: Redis实现分布式锁
date: 2021-05-10
Author: hhw
comments: true
toc: true
tags:  [Redis,Lock]
---

本文记录分布式锁学习

- 实现分布式锁的3种方式
- Redis实现分布式锁需要注意的问题
- 使用Redisson实现分布式锁&可重入锁

### 什么是分布式锁

1. 什么是分布式锁？

   为了保证一个方法或属性在高并发情况下的同一时间只能被同一个线程执行，在传统单体应用单机部署的情况下，可以使用并发处理相关的功能进行互斥控制。但是，随着业务发展的需要，原单体单机部署的系统被演化成分布式集群系统后，由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，单纯的应用并不能提供分布式锁的能力。**为了解决这个问题就需要一种跨机器的互斥机制来控制共享资源的访问，这就是分布式锁要解决的问题。**

2. 什么时候需要使用分布式锁？

   举例：服务器集群部署，定时任务只希望执行1次，这时需要使用分布式锁

3. 分布式锁需要具备哪些条件？

   1、在分布式系统环境下，一个方法在同一时间只能被一个机器的一个线程执行；
   2、高可用的获取锁与释放锁；
   3、高性能的获取锁与释放锁；
   4、具备可重入特性；
   5、具备锁失效机制，防止死锁；
   6、具备非阻塞锁特性，即没有获取到锁将直接返回获取锁失败。

   P.S. 可重入锁：可重入就是说某个线程已经获得某个锁，可以再次获取锁而不会出现死锁。

   为什么要用重入锁？一个方法里，外层方法占用了锁，但是里面还有方法要获得锁，如果不是重入锁，程序无法继续运行，陷入死锁，是重入锁就继续执行。[参考](https://blog.csdn.net/w8y56f/article/details/89554060)

   常见的可重入锁有：Synchronzed、ReentrantLock

### 实现分布式锁的3种方式

#### 基于数据库方式实现

1. 基于数据库的实现方式的核心思想是：在数据库中创建一个表，表中包含**方法名**等字段，并在**方法名字段上创建唯一索引**，想要执行某个方法，就使用这个方法名向表中插入数据，成功插入则获取锁，执行完成后删除对应的行数据释放锁。

2. 表结构参考

   ```sql
   DROP TABLE IF EXISTS `method_lock`;
   CREATE TABLE `method_lock` (
     `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
     `method_name` varchar(64) NOT NULL COMMENT '锁定的方法名',
     `remark` varchar(255) NOT NULL COMMENT '备注信息',
     `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE  CURRENT_TIMESTAMP,
     PRIMARY KEY (`id`),
     UNIQUE KEY `uidx_method_name` (`method_name`) USING BTREE
   ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 COMMENT='锁定中的方法';
   ```

3. 当要执行某个方法时，就使用这个方法名向表中插入数据

   ```sql
   INSERT INTO method_lock (method_name, remark) VALUES ('methodName', '测试的methodName');
   ```

   由于我们对 `method_name` 做了UNIQUE约束，所以如果有多个请求同时提交到数据库的话，数据库只会保证一个操作可以成功，那么可以认为操作成功的线程获取了该方法的锁，可以继续执行方法中的内容。

4. 执行完成后，删除数据，即释放锁

   ```sql
   delete from method_lock where method_name ='methodName';
   ```

5. 存在的问题

   1、因为是基于数据库实现的，数据库的可用性和性能将直接影响分布式锁的可用性及性能，所以，数据库需要双机部署、数据同步、主备切换；

   2、<u>不具备可重入的特性</u>，因为同一个线程在释放锁之前，行数据一直存在，无法再次成功插入数据，所以，需要在表中新增一列，用于记录当前获取到锁的机器和线程信息，在再次获取锁的时候，先查询表中机器和线程信息是否和当前机器和线程相同，若相同则直接获取锁；

   3、<u>没有锁失效机制</u>，因为有可能出现成功插入数据后，服务器宕机了，对应的数据没有被删除，当服务恢复后一直获取不到锁，所以，需要在表中新增一列，用于记录失效时间，并且需要有定时任务清除这些失效的数据；

   4、<u>不具备阻塞锁特性</u>，获取不到锁直接返回失败，所以需要优化获取逻辑，循环多次去获取。

   5、在实施的过程中会遇到各种不同的问题，为了解决这些问题，实现方式将会越来越复杂；依赖数据库需要一定的资源开销，性能问题需要考虑。

#### 基于Redis方式实现

1. 优点

   - 高性能
   - 实现起来较为方便

   缺点

   - 不可重入，需要考虑一些问题

2. 实现思想

   获取锁的时候，使用setnx加锁（当key不存在时，才能设置成功），如果加锁失败，则是该锁已经被占用，如果成功，则是获取锁成功。

   获取锁的时候需要设置一个过期时间，超过时间则释放该锁。

3. 示例

   以集群部署下，定时任务为例，集群部署下，定时任务要求只执行一次，此时就需要使用分布式锁。

   这里使用自定义注解 + 切面的方式去为定时任务加锁

   1、定时任务方法如下：包含自定注解：@RedisTryLock

   ```java
   @Scheduled(cron = "${task.mail.order:0 0 1 * * ?}")
   @RedisTryLock(keyName = "lock_name", expireTime = 50)
   public void task() throws ParseException {
    //.... 业务代码
   }
   ```

   2、自定义注解@RedisTryLock定义

   ```java
   @Target(ElementType.METHOD)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   public @interface RedisTryLock {
   
       /**
        * @Description: 锁的有效时间长，单位：秒
        */
       int expireTime() default 10;
   
       /**
        * @Description: 自定义锁的keyName（不用包含namespace，内部已实现）
        */
       String keyName() default "";
   }
   ```

   3、切面，在此实现锁的获取

   加锁核心代码：

   ```java
   // redisTemplate API参考 
   // https://docs.spring.io/spring-data/redis/docs/2.2.4.RELEASE/api/
   // setIfAbsent方法与setnx意义相同
   if (redisTemplate.opsForValue().setIfAbsent(defKey, ip)) {
     // 设置过期时间，这里存在问题，操作不具备原子性
     redisTemplate.expire(defKey, expireTime, TimeUnit.SECONDS);
     log.info("获得分布式锁成功! key:{}", defKey);
     pjp.proceed();
     redisTemplate.delete(defKey);
     log.info("定时任务执行完，释放分布式锁成功,key:{}", defKey);
     return;
   
   }
   Object redisVal = redisTemplate.opsForValue().get(defKey);
   log.info("{}已在{}机器上占用分布式锁，聚类任务正在执行", defKey, redisVal);
   
   
   ```

   完整切面代码：

   ```java
   @Slf4j
   @Aspect
   @Component
   public class RedisTryLockAspect {
     
     	// 使用spring提供的RedisTemplate来操作redis
       @Autowired
       private RedisTemplate<String, String> redisTemplate;
     
       InetAddress addr = null;
   
       @Around("execution(* *.*(..)) && @annotation(com.logink.supply.chain.task.RedisTryLock)")
       public void redisTryLockPoint(ProceedingJoinPoint pjp) throws Exception {
           String defKey = "redis:lock:";
           RedisTryLock annotation = null;
           Method method = null;
           Class aClass = pjp.getTarget().getClass();
           //获取该切点所在方法的名称,每一个使用切面的方法
           String name = pjp.getSignature().getName();
           try {
               //通过反射获得该方法
               method = aClass.getMethod(name);
               //获得该注解
               annotation = method.getAnnotation(RedisTryLock.class);
               //获取注解对应的值keyName，expireTime
               String keyName = annotation.keyName();
               Integer expireTime = annotation.expireTime();
               Assert.isTrue(0 != expireTime, "redis lock's expireTime is null");
               //获取本机ip
               try {
                   addr = InetAddress.getLocalHost();
   
               } catch (UnknownHostException e) {
                   log.error("获取IP失败--》", e);
   
               }
               String ip = addr.getHostAddress();
               // 设置redis key值
   
               defKey = StringUtils.isBlank(keyName) ? defKey + aClass.getDeclaringClass() + method.getName() : defKey + keyName;
               //根据redis 锁的原理判断是否执行成功，设值成功说明其他服务器没有执行定时任务，反则正在执行
               /** 这里操作不具备原子性，先获得锁才去设置过期时间 存在问题
                *
                * 改造方案1：在redis的value存放过期时间，当获取不到锁的时候，取出时间进行比较，如果已经过期，则获取到锁，重新设置锁信息
                *
                * */
               Date date = new Date();
               if (redisTemplate.opsForValue().setIfAbsent(defKey, ip)) {
                   redisTemplate.expire(defKey, expireTime, TimeUnit.SECONDS);
                   log.info("获得分布式锁成功! key:{}", defKey);
                   pjp.proceed();
                   redisTemplate.delete(defKey);
                   log.info("定时任务执行完，释放分布式锁成功,key:{}", defKey);
                   return;
   
               }
               Object redisVal = redisTemplate.opsForValue().get(defKey);
               log.info("{}已在{}机器上占用分布式锁，聚类任务正在执行", defKey, redisVal);
   
           } catch (NoSuchMethodException e) {
               e.printStackTrace();
               log.error("Facet aop failed error {}", e.getLocalizedMessage());
   
           } catch (Throwable throwable) {
               throwable.printStackTrace();
   
           }
   
       }
   }
   ```

4. 对于示例的V2.0改进版本

   示例存在的问题：操作不具备原子性，获取到锁与设置过期时间是两个步骤，如果获取到锁后服务器宕机，这时并没有设置过期时间，那么会造成死锁。

   思路：在redis的value存放过期时间，当获取不到锁的时候，取出时间进行比较，如果已经过期，则获取到锁，重新设置锁信息，可以避免死锁的发生。

   ```java
   Date date = new Date();
   long now = date.getTime();
   
   // 过期时间时间戳
   long time = now + expireTime * 60;
   if (redisTemplate.opsForValue().setIfAbsent(defKey, String.valueOf(time))) {
     redisTemplate.expire(defKey, expireTime, TimeUnit.SECONDS);
     log.info("获得分布式锁成功! key:{}", defKey);
     pjp.proceed();
     redisTemplate.delete(defKey);
     log.info("定时任务执行完，释放分布式锁成功,key:{}", defKey);
     return;
   }
   
   // 如果上一步获取锁失败，则再手动判断锁是否过期
   String expireDate = redisTemplate.opsForValue().get(defKey);
   if (now > Long.parseLong(expireDate)) {
     redisTemplate.delete(defKey);
     log.info("锁已过期，释放中，成功获得分布式锁！");
     redisTemplate.opsForValue().setIfAbsent(defKey, String.valueOf(time));
     redisTemplate.expire(defKey, expireTime, TimeUnit.SECONDS);
     pjp.proceed();
     redisTemplate.delete(defKey);
     log.info("定时任务执行完，释放分布式锁成功,key:{}", defKey);
     return;
   }
   ```

#### 基于ZooKeeper实现

暂略

### Redisson实现分布式锁

1. 基于Redis方式实现存在一些问题，包括原子性、不可重入等问题，而这些问题使用Redisson都可以解决

2. 什么是Redisson？

   > > [Redisson](https://redisson.org/)是架设在[Redis](http://redis.cn/)基础上的一个Java驻内存数据网格（In-Memory Data Grid）。充分的利用了Redis键值数据库提供的一系列优势，基于Java实用工具包中常用接口，为使用者提供了一系列具有分布式特性的常用工具类。使得原本作为协调单机多线程并发程序的工具包获得了协调分布式多机多线程并发系统的能力，大大降低了设计和研发大规模分布式系统的难度。同时结合各富特色的分布式服务，更进一步简化了分布式环境中程序相互之间的协作。

3. 加入Redisson

   1、引入Jar包

   ```xml
   <dependency>
       <groupId>org.redisson</groupId>
       <artifactId>redisson</artifactId>
       <version>3.10.1</version>
   </dependency>
   ```

   2、创建配置文件

   ```java
   @Component
   @Slf4j
   public class RedissonConfig {
   
       private Redisson redisson = null;
       private Config config = new Config();
   
       @Value("${redis.hostAndPort}")
       private String hostAndPort;
   
       @PostConstruct
       private void init() {
           try {
               config.useSingleServer().setAddress(new StringBuilder().append("redis://").append(hostAndPort).toString());
               redisson = (Redisson) Redisson.create(config);
               log.info("Redisson 初始化完成");
           } catch (Exception e) {
               log.error("init Redisson error ",e);
           }
       }
   
       public Redisson getRedisson() {
           return redisson;
       }
   }
   ```

   3、使用

   核心使用方法

   ```java
   // 注入
   @Autowired
   private RedissonConfig redissonConfig;
   
   // 获取redission锁对象
   RLock lock = redissonConfig.getRedisson().getLock(defKey);
   
   // 尝试获取锁 true表示获取成功，其他服务器没有执行定时任务，反则正在执行
   // tryLock(long waitTime, long leaseTime, TimeUnit unit)
   boolean getLock =  lock.tryLock(0, expireTime, TimeUnit.SECONDS);
   
   // 解锁
   lock.unlock();
   ```

   tryLock()方法参数解释：在获取该锁时如果被其他线程先拿到锁就会进入等待，等待`waitTime`时间，如果还没用机会获取到锁就放弃，返回false；若获得了锁，除非是调用`unlock`释放，那么会一直持有锁，直到超过`leaseTime`指定的时间

   完整代码：

   - 在finally中释放锁，保证任务执行成功或失败，都能及时释放锁

   ```java
   // 注入
   @Autowired
   private RedissonConfig redissonConfig;
   
   // ...
   
   // 获取redission，true表示获取成功，其他服务器没有执行定时任务，反则正在执行
   RLock lock = redissonConfig.getRedisson().getLock(defKey);
   boolean getLock = false;
   try {
       // 尝试获取锁
       getLock =  lock.tryLock(0, expireTime, TimeUnit.SECONDS);
       if (getLock) {
           log.info("获得分布式锁成功! key:{}", defKey);
         	// 执行业务方法
           pjp.proceed();
       } else {
           log.info("{}已在机器上占用分布式锁，聚类任务正在执行", defKey);
       }
   } catch (InterruptedException e) {
       log.error("定时任务执行失败:" + e);
   } finally {
       if (Boolean.TRUE.equals(getLock)) {
           Thread.sleep(5000);
           lock.unlock();
           log.info("Redisson分布式锁释放锁:{}", defKey)
       }
   }
   ```

















