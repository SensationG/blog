---
layout: post
title: mySQL&case、when使用
date: 2020-12-21
Author: hhw
toc: true
comments: true
tags:  [mySQL]

---

### mySql & case、when使用

> 在使用排序时，可能会使用到case、when进行优先级判断排序

#### 案例1 按照关联内容优先排序

```sql
order by
  case when a.enroll_end_time >= #{nowTime} then 1 else 0 end desc,
  case when f.dict_label= #{province} then 1 else 0 end desc,
  case when g.dict_label = #{city} then 1 else 0 end desc
```

注：#{} 是mybatis传入的值 

这里使用case、when进行优先级

#### 案例2 根据内容匹配

