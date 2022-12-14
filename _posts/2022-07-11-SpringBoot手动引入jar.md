> springboot手动引入外部jar包示例

## 一、将jar包放在resource下
![image.png](https://cdn.nlark.com/yuque/0/2022/png/27216901/1657507670850-ddd8bf20-dc50-4de3-9a5a-2a0fb151d1ba.png#clientId=ua9ce37ee-308a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=115&id=ufc6b8f6f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=230&originWidth=752&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17554&status=done&style=none&taskId=u7e36e42b-046e-454e-b102-c9819b8093b&title=&width=376)
## 二、在pom中引用

- 要求scope=system
```xml
<dependency>
  <groupId>com.aspose</groupId>
  <artifactId>aspose-words</artifactId>
  <version>18.8</version>
  <scope>system</scope>
  <systemPath>${project.basedir}/src/main/resources/libs/aspose-words-18.8.jar</systemPath>
</dependency>
```
## 三、在maven中配置打包时包含本地jar
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <includeSystemScope>true</includeSystemScope>
      </configuration>
    </plugin>
  </plugins>
</build>
```
