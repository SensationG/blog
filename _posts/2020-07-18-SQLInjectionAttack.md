---
layout: post
title: SQL注入问题以及#与$
date: 2020-07-25
Author: hhw
toc: true
comments: true
tags:  [SQL,MyBatis]
pinned: true
---


## 1.什么是SQL注入

使用编写SQL语句时的疏忽，导致可以通过传入查询参数来改变SQL的语义，从而非法获取或操纵数据库信息

## 2.SQL注入攻击思路

1：寻找到SQL注入的位置

2：判断服务器类型和后台数据库类型

3：针对不同的服务器和数据库特点进行SQL注入攻击

## 3.SQL注入示例

```sql
String sql = "select * from user_table where username=' "+userName+" ' and password=' "+password+" '";

-- 当输入了上面的用户名和密码，上面的SQL语句变成：
SELECT * FROM user_table WHERE username='’or 1 = 1 -- and password='’

-- 分析SQL语句：
-- 条件后面username=”or 1=1 用户名等于 ” 或1=1 那么这个条件一定会成功；

--然后后面加两个-，这意味着注释，它将后面的语句注释，让他们不起作用，这样语句永远都--能正确执行，用户轻易骗过系统，获取合法身份。

-- 这还是比较温柔的，如果是执行
SELECT * FROM user_table WHERE
username='' ;DROP DATABASE (DB Name) --' and password=''
-- 其后果可想而知…

```

## 4.防御

> 但凡有SQL注入漏洞的程序，都是因为程序要接受外部输入的变量或URL传递的参数，并且这个参数或变量会成为SQL语句的一部分。所以对于用户输入的内容，我们要时刻警惕，要有「外部数据不可信任」的原则。

#### 1.检查变量

**通过检查变量的数据类型和格式，来确保传入的数据合法。**

例如数据库里所有的id都是数字，那么就应该在SQL执行前，检查确保变量id是int类型；如果是邮箱，那么就应该进行邮箱校验。总之，在SQL执行之前，进行变量检查，可以很大程度避免SQL注入。

**对于无法确定固定格式的变量，一定要经过特殊符号过滤或转义处理。**

#### 2.绑定变量，预编译

MySQL的mysqli驱动提供了预编译语句的支持，不同的程序语言，都分别有使用预编译语句的方法

实际上，绑定变量使用预编译语句是预防SQL注入的最佳方式，使用预编译的SQL语句语义不会发生改变，在SQL语句中，变量用问号 ? 表示，黑客即使本事再大，也无法改变SQL语句的结构。

## 5.SQL预编译

#### 1.什么是预编译

通常我们的一条sql在db接收到最终执行完毕返回可以分为下面三个过程：

1. 词法和语义解析

2. 优化sql语句，制定执行计划

3. 执行并返回结果

　　我们把这种普通语句称作 Immediate Statements。　　

　　但是很多情况，我们的一条sql语句可能会反复执行，或者每次执行的时候只有个别的值不同（比如query的where子句值不同，update的set子句值不同,insert的values值不同）。
　　如果每次都需要经过上面的词法语义解析、语句优化、制定执行计划等，则效率就明显不行了。

　　所谓预编译语句就是将这类语句中的值用**占位符**替代，可以视为将sql语句模板化或者说参数化，一般称这类语句叫 **Prepared Statements** 或者 Parameterized Statements
　　预编译语句的优势在于归纳为：一次编译、多次运行，省去了解析优化等过程；此外预编译语句能防止sql注入。
　　当然就优化来说，很多时候最优的执行计划不是光靠知道sql语句的模板就能决定了，往往就是需要通过具体值来预估出成本代价。

#### 2.mySQL预编译功能

下面我们来看一下MySQL中预编译语句的使用。
  （1）建表 首先我们有一张测试表t，结构如下所示：

```
mysql> show create table t\G
*************************** 1. row ***************************
       Table: t
Create Table: CREATE TABLE `t` (
  `a` int(11) DEFAULT NULL,
  `b` varchar(20) DEFAULT NULL,
  UNIQUE KEY `ab` (`a`,`b`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

　（2）编译

　　我们接下来通过 `PREPARE stmt_name FROM preparable_stm`的语法来预编译一条sql语句

```
mysql> prepare ins from 'insert into t select ?,?';
Query OK, 0 rows affected (0.00 sec)
Statement prepared
```

　（3）执行

　　我们通过`EXECUTE stmt_name [USING @var_name [, @var_name] ...]`的语法来执行预编译语句

```
mysql> set @a=999,@b='hello';
Query OK, 0 rows affected (0.00 sec)
 
mysql> execute ins using @a,@b;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0
 
mysql> select * from t;
+------+-------+
| a    | b     |
+------+-------+
|  999 | hello |
+------+-------+
1 row in set (0.00 sec)
```

　　可以看到，数据已经被成功插入表中。

　　MySQL中的预编译语句作用域是session级，但我们可以通过max_prepared_stmt_count变量来控制全局最大的存储的预编译语句。

```
mysql> set @@global.max_prepared_stmt_count=1;
Query OK, 0 rows affected (0.00 sec)
 
mysql> prepare sel from 'select * from t';
ERROR 1461 (42000): Can't create more than max_prepared_stmt_count statements (current value: 1)
```

当预编译条数已经达到阈值时可以看到MySQL会报如上所示的错误。

  （4）释放
　　如果我们想要释放一条预编译语句，则可以使用`{DEALLOCATE | DROP} PREPARE stmt_name`的语法进行操作:

```
mysql> deallocate prepare ins;
Query OK, 0 rows affected (0.00 sec)
```

## 6.为什么PrepareStatement可以防止sql注入

> **原理是采用了预编译的方法，先将SQL语句中可被客户端控制的参数集进行编译，生成对应的临时变量集，再使用对应的设置方法，为临时变量集里面的元素进行赋值，赋值函数setString()，会对传入的参数进行强制类型检查和安全检查，所以就避免了SQL注入的产生。下面具体分析**

　**（1）为什么Statement会被sql注入**

　　因为Statement之所以会被sql注入是因为SQL语句结构发生了变化。比如：

```
"select*from tablename where username='"+uesrname+  
"'and password='"+password+"'"
```

　　在用户输入'or true or'之后sql语句结构改变。

```
select*from tablename where username=''or true or'' and password=''
```

　　这样本来是判断用户名和密码都匹配时才会计数，但是经过改变后变成了或的逻辑关系，不管用户名和密码是否匹配该式的返回值永远为true;

　**（2）为什么Preparement可以防止SQL注入。**

　　因为Preparement样式为

```
select*from tablename where username=? and password=?
```

　　该SQL语句会在得到用户的输入之前先用数据库进行预编译，这样的话不管用户输入什么用户名和密码的判断始终都是并的逻辑关系，防止了SQL注入

　　简单总结，参数化能防注入的原因在于，语句是语句，参数是参数，参数的值并不是语句的一部分，数据库只按语句的语义跑，至于跑的时候是带一个普通背包还是一个怪物，不会影响行进路线，无非跑的快点与慢点的区别。

## 7.mybatis防止SQL注入

**mybatis中的#和$的区别：**

1. `#{}` **使用？占位符，并且传入的数据都当成一个字符串，会对自动传入的数据加一个双引号。**
   如：where username=#{username}，如果传入的值是111,那么解析成sql时的值为where username="111", 如果传入的值是id，则解析成的sql为where username="id"。
2. `${}`将传入的**数据直接显示生成在sql**中。
   如：`where username=${username}`，如果传入的值是111,那么解析成sql时的值为where username=111；
   如果传入的值是;drop table user;，则解析成的sql为：select id, username, password, role from user where username=;drop table user;
3. #方式能够很大程度防止sql注入，`$`方式无法防止Sql注入。
4. `$`方式一般用于传入数据库对象，例如传入表名.
5. 一般能用#的就别用​`$`，若不得不使用“`$`{xxx}”这样的参数，要手工地做好过滤工作，来防止sql注入攻击。
6. 在MyBatis中，“`${xxx}`”这样格式的参数会直接参与SQL编译，从而不能避免注入攻击。但涉及到动态表名和列名时，只能使用“`${xxx}`”这样的参数格式。所以，这样的参数需要我们在代码中手工进行处理来防止注入。

【结论】在编写MyBatis的映射语句时，尽量采用“#{xxx}”这样的格式。若不得不使用“${xxx}”这样的参数，要手工地做好过滤工作，来防止SQL注入攻击。

**mybatis是如何做到防止sql注入的**

　　MyBatis框架作为一款半自动化的持久层框架，其SQL语句都要我们自己手动编写，这个时候当然需要防止SQL注入。其实，MyBatis的SQL是一个具有“**输入+输出**”的功能，类似于函数的结构，参考上面的两个例子。其中，parameterType表示了输入的参数类型，resultType表示了输出的参数类型。回应上文，如果我们想防止SQL注入，理所当然地要在输入参数上下功夫。上面代码中使用#的即输入参数在SQL中拼接的部分，传入参数后，打印出执行的SQL语句，会看到SQL是这样的：

```sql
select id, username, password, role from user where username=? and password=?
```

　　不管输入什么参数，打印出的SQL都是这样的。这是因为MyBatis启用了预编译功能，在SQL执行前，会先将上面的SQL发送给数据库进行编译；执行时，直接使用编译好的SQL，替换占位符 **“?”** 就可以了。**因为SQL注入只能对编译过程起作用，所以这样的方式就很好地避免了SQL注入的问题**。

　　【底层实现原理】MyBatis是如何做到SQL预编译的呢？其实在框架底层，是JDBC中的PreparedStatement类在起作用，PreparedStatement是我们很熟悉的Statement的子类，它的对象包含了编译好的SQL语句。这种“准备好”的方式不仅能提高安全性，而且在多次执行同一个SQL时，能够提高效率。原因是SQL已编译好，再次执行时无需再编译。**因为SQL注入仅发生在编译时。**

 [参考](https://www.cnblogs.com/myseries/p/10821372.html)

 
