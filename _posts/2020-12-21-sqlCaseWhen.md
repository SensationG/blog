---
layout: post
title: mySQL复杂函数使用&日期使用
date: 2020-12-21、2021-03-24 [更新]
Author: hhw
toc: true
comments: true
tags:  [mySQL]
---

#### 1. case、when使用

> 在使用排序时，可能会使用到case、when进行优先级判断排序

**案例1 按照关联内容优先排序**

```sql
order by
  case when a.enroll_end_time >= #{nowTime} then 1 else 0 end desc,
  case when f.dict_label= #{province} then 1 else 0 end desc,
  case when g.dict_label = #{city} then 1 else 0 end desc
```

注：#{} 是mybatis传入的值 

这里使用case、when进行优先级

#### 2. 查询日期是今天

```sql
-- go.order_time 为示例，根据实际情况从业务表中取
to_days(go.order_time) = to_days(now())
```

#### 3. 查询最近七日

```sql
-- 包括今日起 往前推6日
TO_DAYS(NOW()) - TO_DAYS(order_time) = 6
-- 包括今日起 往前推5日 
TO_DAYS(NOW()) - TO_DAYS(order_time) = 5
-- 昨日 
TO_DAYS(NOW()) - TO_DAYS(order_time) = 1

-- 如果是x日内，则可以用 <= 表示 示例：3日内
TO_DAYS(NOW()) - TO_DAYS(order_time) <= 3
```

#### 4. 查询本周内数据

- 使用`YEARWEEK()`函数来查询今日是几年的第几周，默认一周起始日为周天
- 使用`YEARWEEK(日期,1) `使用参数 1 来修改一周起始日为周一

```sql
-- 查询本周数据 例如今日是周二 则查询周一～周日的数据 order_time为示例，根据业务情况取
WHERE YEARWEEK(date_format(order_time,'%Y-%m-%d'),1) = YEARWEEK(now(),1)
-- 查询上周数据
WHERE YEARWEEK(date_format(order_time,'%Y-%m-%d'),1) = YEARWEEK(now(),1) - 1
-- 查询上上周
WHERE YEARWEEK(date_format(order_time,'%Y-%m-%d'),1) = YEARWEEK(now(),1) - 2
```

#### 5. 查询本周的起始/结束日期

- `subdate()` 函数，对日期进行减运算
- `WEEKDAY()`函数，返回date的星期索引（计算当前日期所在一周内的index，周一为0，周二为1.....）

```sql
-- 查询本周的开始日期(周一)和结束日期(周日)
select subdate(CURDATE(),WEEKDAY(CURDATE())) as 起始, 
			 subdate(CURDATE(),WEEKDAY(CURDATE()) - 6) as 结束;
-- 查询上周的开始日期和结束日期
SELECT subdate(CURDATE(),WEEKDAY(CURDATE()) + 7) as 起始,
       subdate(CURDATE(),WEEKDAY(CURDATE()) + 1) as 结束,
-- 查询上上周的开始日期和结束日期
SELECT subdate(CURDATE(),WEEKDAY(CURDATE()) + 14) as 起始,
       subdate(CURDATE(),WEEKDAY(CURDATE()) + 8) as 结束,
-- 查询上上上周的开始日期和结束日期
SELECT subdate(CURDATE(),WEEKDAY(CURDATE()) + 21) as 起始,
       subdate(CURDATE(),WEEKDAY(CURDATE()) + 15) as 结束,
```

#### 6. 查询本月的起始/结束日期

```sql
-- 获取当前月第一天
select date_add(date_add(last_day(now()),interval 1 day),interval -1 month);
-- 获取当前月最后一天
select last_day(NOW());
```

#### 7. 对查询结果显示序号

- 使用 自定义参数的方式

示例：

```sql
SELECT ro.des_province_name as smc, count(1) as jsymydsl, @row_num := @row_num + 1 as px
FROM go_order go
        JOIN req_order ro on ro.go_order_id = go.go_order_id,
        (
            SELECT @row_num := 0
        ) as row
WHERE go.scheduling_type = 'DV'
    AND go.order_status <> '2'
    AND go.is_deleted <> 1
GROUP BY ro.des_province_code
ORDER BY px;
```

#### 8. COALESCE 空值处理

- `COALESCE(a1,b1)` 空值处理函数，如果a1为空，则返回b1

- 也可以填写多个参数连用：

  ```sql
  COALESCE ( expression,value1,value2……,valuen) 
  -- 如果expression不为空值则返回expression；否则判断value1是否是空值，
  -- 如果value1不为空值则返回value1；否则判断value2是否是空值，
  -- 如果value2不为空值则返回value2；……以此类推，
  -- 如果所有的表达式都为空值，则返回NULL。 
  ```

#### 9. IF 函数

- 使用IF函数来进行判断

- `IF(表达式,a1,b1)`如果表达式为true，则返回a1，否则返回a2

- 示例

  ```sql
  -- 如果创建时间在本月，则返回1 ，否则返回0
  IF(tr.create_time between date_add(date_add(last_day(now()), interval 1 day), interval -1 month) AND last_day(NOW()),'1', '0')       
  ```

#### 10. group_concat

- 使用group_concat将结果集进行合并，将选择的字段合并到一个字段进行显示，且默认用逗号进行隔开

```sql
select t.tra_order_no, group_concat(t.sum, mdc.dict_name)
from (
         select tr.tra_order_no, gc.package_type, SUM(gc.cargo_package) as sum
         from tra_order tr
                  join go_cargo gc on gc.business_id = tr.tra_order_id AND gc.business_type = 'traOrder'
         group by tr.tra_order_no, gc.package_type
     ) t
         join md_dict_code mdc on mdc.dict_code = t.package_type and mdc.dict_type_code = 'LOGINK_PACKAGE_TYPE'
group by t.tra_order_no;
```















