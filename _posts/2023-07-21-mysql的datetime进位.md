---
layout: post
title: MySQL中datetime和timestemp类型23:59:59秒入库后会变成下一天的00:00:00
date: 2023-07-21
Author: hhw
comments: true
toc: true
tags:  [java,mysql]

---

## 1.问题产生

业务中需要记录一个单据的的开始时间和截止时间，对应的是MySQL表中的datetime类型。java生成的截止时间是2022-09-20 23:59:59 ，但是Mybatis-plus入库了之后查看数据库却变成第二天的00:00:00，即：2023-09-20 23:59:59 变成了 2023-09-21 00:00:00。

## 2.问题原因

MySQL数据库对于毫秒大于500的数据进行进位

## 3.解决

采用Hutool工具包，方便的进行时间的管理和转换，DateUtil Hutool官网将生成的时间往前偏移999毫秒即可。

`示例：DateUtil.endOfDay(DateUtil.date()).offset(DateField.MILLISECOND, -999)`







