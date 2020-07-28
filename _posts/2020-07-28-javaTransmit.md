---

layout: post
title: java是值传递还是引用传递？
date: 2020-07-28
Author: hhw
tags:  [java]

---

将参数传递给方法有两种常见的方式，一种是“值传递”，一种是“引用传递”。C 语言本身只支持值传递，它的衍生品 C++ 既支持值传递，也支持引用传递。

### 基本类型与引用类型

<img src="https://raw.githubusercontent.com/SensationG/images/master/20200728162839.png" alt="image-20200728151855902" style="zoom:50%;" />

```java
int num = 10;
String str = "hello";
```

num是基本类型，值就直接保存在变量中。

str是引用类型，变量中保存的只是实际对象的地址。一般称这种变量为"引用"，引用指向实际对象，实际对象中保存着内容。

>  值和引用存储在 stack（栈）中，而对象存储在 heap（堆）中。

<img src="https://raw.githubusercontent.com/SensationG/images/master/20200728162856.png" alt="image-20200728152626805" style="zoom:67%;" />

之所以有这个区别，是因为：

栈的优势是，存取速度比堆要快，仅次于直接位于 CPU 中的寄存器。但缺点是，栈中的数据大小与生存周期必须是确定的。
堆的优势是可以动态地分配内存大小，生存周期也不必事先告诉编译器，Java 的垃圾回收器会自动收走那些不再使用的数据。但由于要在运行时动态分配内存，存取速度较慢。

### 赋值运算'='的区别

```java
num = 20;
str = "java";
```

对于基本类型，赋值运算符会直接改变变量的值，原来的值会被覆盖掉。

对于引用类型，赋值运算符会改变引用所指向的地址，而原对象并不会改变。（没有被任何引用所指向的对象是垃圾，会被垃圾回收器回收）

### 基本类型的参数传递

Java 有 8 种基本数据类型，分别是 int、long、byte、short、float、double 、char 和 boolean。它们的值直接存储在栈中，每当作为参数传递时，都会将原始值（实参）复制一份新的出来，给形参用。形参将会在被调用方法结束时从栈中清除。

示例代码：

```java
public class PrimitiveTypeDemo {
    public static void main(String[] args) {
        int age = 18;
        modify(age);
        System.out.println(age);
    }

    private static void modify(int age1) {
        age1 = 30;
    }
}
```

- main方法的age是基本类型，直接存储在栈中。
- 调用 `modify()` 方法时，将为实参age创建一个副本（形参age1），即在栈中创建一个age的副本，并且它的值也为18
- 对型参age1的任何修改都不会影响到实参age

<img src="https://raw.githubusercontent.com/SensationG/images/master/20200728162901.png" alt="image-20200728153253489" style="zoom:50%;" />

### 引用类型的参数传递

创建一个实体类：

```java
public class User {
  
    private String name;
  
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

示例代码：

```java
public class Demo {
    public static void main(String[] args) {
      
        User user = new User();
        
        user.setName("hhw");
        System.out.println(user.getName()); // hhw

        modify(user);
        System.out.println(user.getName()); //ymd
    }

    public static void modify(User user1) {
        user1.setName("ymd");
    }

}
```

输出结果：

```
hhw
ymd
```

- 在`modify()`时，将为实参user创建一个副本（user1），即在栈中创建一个user副本user1，他们都指向堆中的同一个user对象。

  <img src="https://raw.githubusercontent.com/SensationG/images/master/20200728162908.png" alt="image-20200728155143621" style="zoom:50%;" />

- 在`modify()`方法中，修改了型参user1的name属性为"ymd"，由于user1指向的对象也是user，所以引用user的name也将变为"ymd"。

- 因此最后输出的结果中user的name属性值会变为"ymd"。

### 附加

将modify的代码进行修改，将user1指向一个新对象，那么实参的user会改变吗？

示例：

```java
public class Demo {
    public static void main(String[] args) {
      
        User user = new User();
        
        user.setName("hhw");
        System.out.println(user.getName()); 

        modify(user);
        System.out.println(user.getName()); 
    }

    public static void modify(User user1) {
        user1 = new User(); // 
        user1.setName("ymd");
    }

}
```

输出结果：

```
hhw
hhw
```

- 与上面相似的，在调用`modify()`方法的时候会为引用user在栈中创建一个副本（user1），他们都指向同一个user对象。

- 在`modify()`方法中，形参user1指向了一个新的对象，随后将name属性的值设置为"ymd"

  <img src="https://raw.githubusercontent.com/SensationG/images/master/20200728162912.png" alt="image-20200728160812925" style="zoom:50%;" />

  当user1指向一个新对象时，引用user1指向的对象会由原先的user指向一个新的user对象，并将该新对象的name值设置为"ymd"

- 修改user1的name时，user没有受到影响，是因为他们所指向的对象不同，修改不受影响。

### 结论

网络上对这一块争议还是比较大的。

对于基本数据类型，基本都默认是值传递，在栈中新建了一个变量。

对于引用数据类型，在栈中存储堆中某个对象的一个引用，那么对象作为参数传递，到底是值传递还是引用传递？个人观点是，如果认为传递的是堆中某个对象的引用，它是一个值（如内存地址），那可以理解为值传递；如果认为传递的就是对象的引用，那就是引用传递。

[参考](https://blog.csdn.net/qing_gee/article/details/105650149?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)

