---
layout: post
title: Spring Cache使用记录
date: 2021-12-13
Author: hhw
comments: true
toc: true
tags:  [java,cache,redis]
---
## Spring Cache缓存使用

### 1.配置

可以配置通过Redis缓存或通过本地缓存，此处以配置本地缓存为例，区别是本地缓存无法设置过期时间，Redis可设置过期时间。

新建配置类，配置Cache走本地缓存：

```java
@Configuration
public class CacheManagerConfig {

    /**
     * 自定义缓存管理器
     *
     * @return
     * @Author huanghwh
     * @Date 2021/12/9 17:22
     */
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        // 指定@Cacheable缓存位置，使用内存进行缓存；指定@Cacheable的名称，要求value等于该值才能正确命中
        cacheManager.setCaches(Arrays.asList(new ConcurrentMapCache("epaasUserCache")));
        cacheManager.afterPropertiesSet();
        return cacheManager;
    }

}
```

### 2.使用

#### 1.@Cacheable

> 对于使用@Cacheable标注的方法，Spring在每次执行前都会检查Cache中是否存在相同key的缓存元素，如果存在就不再执行该方法，而是直接从缓存中获取结果进行返回，否则才会执行并将返回结果存入指定的缓存中

作用于方法上，用来执行缓存，通常需要指定value和key。

- value：cache名称；

- key：指定Spring缓存方法的返回结果时对应的key的，是否走缓存的判断条件。如果入参是对象，可采用`"#user.id"`的形式。

- condition：指定缓存发生的条件，condition属性默认为空，表示将缓存所有的调用情形

  `@Cacheable(value={“users”}, key=”#user.id”, condition=”#user.id%2==0”)`

一组使用案例：

```java
@Cacheable(value = "epaasUserCache", key = "#userGuid")
@Override
public EpaasUserDto getEpaasUserByUserGuid(String userGuid) { 
    //... 
}
```

#### 2.@CacheEvict

作用于方法上，用来清除缓存，需指定value和key（与@Cacheable类似）

- value：与@Cacheable相同
- key：与@Cacheable相同
-  allEntries：boolean，默认false，是否清除所有缓存中的元素
- beforeInvocation：boolean，默认false，即清除缓存是在方法执行后触发。设置为true后，Spring会在调用该方法之前清除缓存中的指定元素。

```JAVA
@CacheEvict(value = "epaasUserCache", key = "#epaasUserRole.userGuid")
@Transactional(rollbackFor = Exception.class)
@Override
public boolean saveUserRole(EpaasUserRole epaasUserRole) {
	// ...
}
```

#### 3.@CachePut

> @CachePut也可以声明一个方法支持缓存功能。与@Cacheable不同的是使用@CachePut标注的方法在执行前不会去检查缓存中是否存在之前执行过的结果，<font color="blue">而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中。</font>
>
>  @CachePut也可以标注在类上和方法上。使用@CachePut时我们可以指定的属性跟@Cacheable是一样的。  

