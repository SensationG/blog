

Spring 为事务管理提供了丰富的功能支持。Spring 事务管理分为编程式和声明式的两种方式。编程式事务指的是通过编码方式实现事务；声明式事务**基于 AOP**,将具体业务逻辑与事务处理解耦。声明式事务管理使业务代码逻辑不受污染, 因此在实际使用中声明式事务用的比较多。声明式事务有两种方式，一种是在配置文件（xml）中做相关的事务规则声明，另一种是基于 @Transactional 注解的方式。

**需要明确声明式事物的前提：**

- 默认配置下Spring只会回滚 **运行时异常**（继承自RuntimeException）或 **Error**；（[参考这里](https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/htmlsingle/#transaction-declarative-rolling-back)）
- @Transactional注解只能应用到public方法才有效；[参考这里 Method visibility and @Transactional](https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/htmlsingle/#transaction-declarative-annotations)
- 使用：只需简单的加上 `@Transactional`注解即可， spring boot 会[自动配置](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-auto-configuration)一个 `DataSourceTransactionManager`，可配置属性

**异常复习：**

![image-20200705143919342](https://raw.githubusercontent.com/SensationG/images/master/note/image-20200705143919342.png)

如果不对运行时异常进行捕获，那么出现运行时异常后将会终止线程或主程序，此时事物会发生回滚，如果不想终止，可以使用catch进行异常捕获；捕获后，接下来的代码将继续正常执行，事务不发生回滚；

非运行时异常（除RuntimeException以外），如IOException，SQLException等等，必须使用catch进行捕获，否则程序就无法通过；因为异常被捕获，所以无法发生回滚；

##### 事务的传播级别

通过配置`propagation` 属性，可以选择事务的传播行为

可选的值有：

- Propagation.REQUIRED（默认）

  如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。

  使用示例：

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  @Override
  public int insertExample(AccompanyRecord accompanyRecord)
  {
      Student student = new Student();
    
    	// myabtis接口
      studentMapper.updateAttention(student);
      accompanyRecordMapper.insertAccompanyRecord(accompanyRecord);
    
      if (true) {
        // 抛异常 回滚
        throw new RuntimeException();
      }
  }
  ```

  以上两条sql执行完后，又抛出了运行时异常，所以上面两条sql都会进行回滚；

- Propagation.SUPPORTS

  如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。

- Propagation.MANDATORY

  如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。

- Propagation.REQUIRES_NEW

  重新创建一个新的事务，如果当前存在事务，暂停当前的事务。

- Propagation.NOT_SUPPORTED

  以非事务的方式运行，如果当前存在事务，暂停当前的事务。

- Propagation.NEVER

  以非事务的方式运行，如果当前存在事务，则抛出异常。

- Propagation.NESTED

  和 Propagation.REQUIRED 效果一样。

**事务使用的注意事项：**

在默认的代理模式下，只有目标方法由外部调用，才能被 Spring 的事务拦截器拦截。在同一个类中的两个方法直接调用，是不会被 Spring 的事务拦截器拦截；【参照Spring官方文档】

示例：

在有需求如下，就算 save 方法的后面抛异常了，也不能影响 method1 方法的数据插入。直接给 method1 页加入一个新的事务，这样 method1 就会在这个新的事务中执行，原来的事务不会影响到新的事务。比如 method1 方法上面再加入注解 @Transactional，设置 propagation 属性为 Propagation.REQUIRES_NEW，代码如下。

```java
@Transactional(propagation = Propagation.REQUIRED)
@Override
public void save() {

    method1();

    User user = new User("服部半藏");
    userMapper.insertSelective(user);

    if (true) {
        throw new RuntimeException("save 抛异常了");
    }
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void method1() {
    User user = new User("宫本武藏");
    userMapper.insertSelective(user);
}
```

此时会发现，即使抛出异常了，仍然发生回滚；通过日志可以发现，从头到尾都只有一个事物，并没有创建新事物；所以我们要想创建新事物，必须新建一个类，将该方法由外部调用，也就是新建一个类用于存放method1即可；此时会发现可以按照需求执行；

流程：@Transactional 的 propagation 属性为 Propagation.REQUIRES_NEW ，所以接着暂停了 save 方法的事务，重新创建了xxx.method1 方法的事务，接着 xxx.method1 方法的事务提交，接着 save 方法的事务回滚。



##### 事务的隔离级别

通过配置`isolation` 属性，可以选择事务的隔离级别

可选的值有：

- Isolation.DEFAULT（默认）

  使用底层数据库默认的隔离级别。

- Isolation.READ_UNCOMMITTED

- Isolation.READ_COMMITTED

- Isolation.REPEATABLE_READ

- Isolation.SERIALIZABLE

##### @tansactional注解属性

- timeout 属性
  事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

- readOnly 属性
  指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。

- **rollbackFor 属性**
  用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。

- noRollbackFor 属性
  抛出指定的异常类型，不回滚事务，也可以指定多个异常类型。


注：alibaba规范必须显示声明RollBackFor：

1. 让checked例外也回滚：在整个方法前加上 @Transactional(rollbackFor=Exception.class)

2. 让unchecked例外不回滚： @Transactional(notRollbackFor=RunTimeException.class)

3. 不需要事务管理的(只查询的)方法：@Transactional(propagation=Propagation.NOT_SUPPORTED)

注意： 如果异常被try｛｝catch｛｝了，事务就不回滚了，如果想让事务回滚必须再往外抛try｛｝catch｛throw Exception｝。

**事务实现机制**

基于AOP，暂略
