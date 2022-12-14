## 一、简介

> 解决的问题


用来存当前线程的局部变量，不同线程间互不干扰。拿完数据记得需要移除数据，不然JVM不会将ThreadLocal回收(可能还会被引用)，多了就会出现内存泄漏的情况。

## 二、使用

### 1.简单的使用

> 每次需要显式的为`threadLocal`赋值，并在线程结束时显式的销毁


#### 初始化`ThreadLocal`

```java
import org.springframework.stereotype.Component;
import java.util.Map;

@Component
public class ThreadLocalConfig {

    // jdk建议将 ThreadLocal 定义为 private static ，这样就不会有弱引用，内存泄漏的问题了
    private final static ThreadLocal<Map> mapThreadLocal = new ThreadLocal<>();


    //获取当前线程的存的变量
    public Map get() {
        return mapThreadLocal.get();
    }

    //设置当前线程的存的变量
    public void set(Map map) {
        this.mapThreadLocal.set(map);
    }

    //移除当前线程的存的变量
    public void remove() {
        this.mapThreadLocal.remove();
    }

}
```

#### 使用

```java
@Autowired
ThreadLocalConfig threadLocalConfig;
//....

// 赋值
Map map = new HashMap<String, String>();
map.put("userId", userId);
threadLocalConfig.set(map);

// 获取
Map map = threadLocalConfig.get();
String userId = map.get("userId");

// 移除
threadLocalConfig.remove();
```

### 2.配合拦截器使用

> 通过拦截器对threadLocal进行赋值，并在线程结束时通过拦截器销毁
>  
> 使用场景：当前用户信息
>  
> 思路：访问接口时解析获取用户信息，然后将当前用户信息放入`ThreadLocal`；业务执行过程中可随时取用；访问结束时候删除`ThreadLocal`中信息（线程放入线程池并不一定会销毁）


#### 配置User专用的`ThreadLocal`

```java
@Component
public class ThreadLocalUtil {

    /**
      * 保存用户对象的ThreadLocal  在拦截器操作 添加、删除相关用户数据
      */
    private static final ThreadLocal<User> userThreadLocal = new ThreadLocal<>();

    /**
      * 添加当前登录用户方法  在拦截器方法执行前调用设置获取用户
      * @param user
      */
    public void addCurrentUser(FeginUser user){
        userThreadLocal.set(user);
    }

    /**
      * 获取当前登录用户方法
      */
    public FeginUser getCurrentUser(){
        return userThreadLocal.get();
    }


    /**
      * 删除当前登录用户方法  在拦截器方法执行后 移除当前用户对象
      */
    public void remove(){
        userThreadLocal.remove();
    }
}
```

#### 拦截器

```java
@Component
@Slf4j
public class UserInfoInterceptor implements HandlerInterceptor  {

    // 当前登录用户查询工具  todo：更换成自己系统的当前用户查询方法
    @Autowired
    private UserInfoUtil userInfoUtil;

    /**
     * 请求执行前执行的，将用户信息放入ThreadLocal
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        User user;
        try{
             user = userInfoUtil.getUser();
        } catch (CustomException e){
            log.info("***************用户未登录， ThreadLocal无信息****************");
            return true;
        }
        if (null != user) {
            log.info("*******************用户已登录，用户信息放入ThreadLocal***************");
            ThreadLocalUtil.addCurrentUser(user);
            return true;
        }
        log.info("**********用户未登录， ThreadLocal无信息************");
        return true;
    }

    /**
     * 接口访问结束后，从ThreadLocal中删除用户信息
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("******接口调用结束， 从ThreadLocal删除用户信息********");
        ThreadLocalUtil.remove();
    }
}
```

#### 注册拦截器

```java
@Configuration
@ComponentScan
public class MyAppConfigurer extends WebMvcConfigurationSupport {

    @Autowired
    private UserInfoInterceptor userInfoInterceptor;

    /**
     * 拦截器，将用户信息放入threadLocal
     */
    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(this.userInfoInterceptor).addPathPatterns("/**");
        super.addInterceptors(registry);
    }
}
```

#### 使用

> 使用方法同1.简单使用


## 三、误区解答

> 一个线程对应一个ThreadLocal，重复设置值会被覆盖


一个线程其实可以设置多个与当前线程相关的数据，只是要设置多个ThreadLocal。若只在一个ThreadLocal里面重复设置是会被覆盖的。

也就是说如果要存放多个与线程相关的对象只需要多new几个ThreadLocal，每个ThreadLocal对应一个数据对象，每个数据对象的key就是各个ThreadLocal本身。

> ThreadLocal就是一个map，key就是线程相关的参数，value就是要存放的数据


ThreadLocal的set方法其实就是把自己作为map的key，数据作为value设置到了当前线程对象的成员变量threadLocal里面，而这个threadLocal成员变量其实就是一个非常类似于HashMap的map类型对象，这个对象会在第一次set方法被调用时候初始化。

## 四、工具类附录
```json
import java.util.*;

/**
 * @Author: huanghwh
 * @Date: 2022/04/20 16:30
 * @Description: ThreadLocalUtil 线程工具类
 */
public class ThreadLocalUtil<T> {

    private static final ThreadLocal<Map<String, Object>> threadLocal = new ThreadLocal() {
        @Override
        protected Map<String, Object> initialValue() {
            return new HashMap<>(4);
        }
    };


    public static Map<String, Object> getThreadLocal(){
        return threadLocal.get();
    }

    public static <T> T get(String key) {
        Map map = (Map)threadLocal.get();
        return (T)map.get(key);
    }

    public static <T> T get(String key,T defaultValue) {
        Map map = (Map)threadLocal.get();
        return (T)map.get(key) == null ? defaultValue : (T)map.get(key);
    }

    public static void set(String key, Object value) {
        Map map = (Map)threadLocal.get();
        map.put(key, value);
    }

    public static void set(Map<String, Object> keyValueMap) {
        Map map = (Map)threadLocal.get();
        map.putAll(keyValueMap);
    }

    public static void remove() {
        threadLocal.remove();
    }

    public static <T> Map<String,T> fetchVarsByPrefix(String prefix) {
        Map<String,T> vars = new HashMap<>();
        if( prefix == null ){
            return vars;
        }
        Map map = (Map)threadLocal.get();
        Set<Map.Entry> set = map.entrySet();

        for( Map.Entry entry : set){
            Object key = entry.getKey();
            if( key instanceof String ){
                if( ((String) key).startsWith(prefix) ){
                    vars.put((String)key,(T)entry.getValue());
                }
            }
        }
        return vars;
    }

    public static <T> T remove(String key) {
        Map map = (Map)threadLocal.get();
        return (T)map.remove(key);
    }

    public static void clear(String prefix) {
        if( prefix == null ){
            return;
        }
        Map map = (Map)threadLocal.get();
        Set<Map.Entry> set = map.entrySet();
        List<String> removeKeys = new ArrayList<>();

        for( Map.Entry entry : set ){
            Object key = entry.getKey();
            if( key instanceof String ){
                if( ((String) key).startsWith(prefix) ){
                    removeKeys.add((String)key);
                }
            }
        }
        for( String key : removeKeys ){
            map.remove(key);
        }
    }
}
```
