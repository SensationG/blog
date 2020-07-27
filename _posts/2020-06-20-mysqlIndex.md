---
layout: post
title: MySQL索引概要以及失效问题
date: 2020-06-20
Author: hhw
toc: true
comments: true
tags:  [MySQL]
---

本文根据mysql的索引进行总结；<br>

> 先来介绍下索引：<br>

**定义**：高效获取数据的数据结构。索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录<br>
**目的**：提高查找效率。简单理解为 排好序的快速查找数据结构。索引类型：一般是bTree索引(数据结构本文先不作解释)<br>
**影响到2部分功能**：
1. where （通常是where）
2. order by 使用索引增加速度
3. 所以在创建索引时，你需要确保该索引是应用在 SQL 查询语句的条件(一般作为 WHERE 子句的条件)。

**劣势**：
1. 索引虽然能增加查询速度，但是也会使得update，insert，delete的速度变慢，性能会收到影响，因为在做增删改的同时，还要更新索引信息
2. 索引也是一张表，保存主键和索引字段，需要占空间

## 1.索引类型
索引分为五类：普通索引；唯一索引；主键索引；组合索引；全文索引；
> 普通索引 NORMAL
- 无任何限制，用于加速查询
- 可设置索引长度<br>
为何要设置索引长度？譬如建表时，Name 长度为 50，但是我们只设置10就足够了，因为一般情况下名字的长度不会超过 10，这样会加速索引查询速度，还会减少索引文件的大小，提高 INSERT 的更新速度。

> 唯一索引 UNIQUE
- 索引列的值必须唯一，但允许为null。
- 如果是组合索引，那么列值的组合必须唯一。

> 主键索引 PRIMARY KEY
- 特殊的唯一索引，一张表只能有一个主键，不允许null，一般在建表时同时创建主键索引。
> 组合索引
- 指在多个字段上创建的索引，索引类型可以是以上三种类型。
- 最左前缀原则：只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。<br>
举例：<br>
我们在三个栏位上建立了索引`(name,age,city)`
    ```sql
    -- 会用到
    select * from table where name='aa';
    select * from table where name='aa' and age=10;
    select * from table where name='aa' and city='bj';
    -- 不会用到
    select * from table where age=10 and city='bj';
    select * from table where city='bj';
    ```
> 全文索引 FULLTEXT
- 用来找文本中的关键字，与以上索引均不相同，需要配合match against操作使用，而不是一般的where语句加like。
- 只有在char，varchar，text列可建立

> 空间索引 SPATIAL （暂时略）

## 2.索引失效

> mysql什么时候走索引？

- 在使用 <，<=，=，>，>=，BETWEEN，IN(视情况而定)，以及某些时候的LIKE才会使用索引。因为Like在以通配符%和_开头作查询时，MySQL不会使用索引。
- 聚集函数Min(),Max()中的字段
- order by 和 Group By中的字段

> mysql什么时候不走索引？

- MySQL能估计出全表扫描比使用索引更快时，不使用索引；譬如当表字段太少的时候。
- like前使用通配符的时候
- 在索引列进行函数操作的
- 使用not in，or(不确定)，<>，!=等

> 如何判断sql有没有走索引？

在sql语句前加上 explain 例如：

`explain select * from test where a<10 and c <10;`
通过查看type字段来判断是否有走索引
