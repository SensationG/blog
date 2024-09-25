---
layout: post
title: 使用Stream处理集合
date: 2021-08-23
Author: hhw
comments: true
toc: true
tags:  [java]
---

## 1. List Stream常用示例

**1.条件筛选出List**

```java
 List<CollectSummaryCost> parentItemList = summaryList.stream().filter(summaryCost -> Objects.equals(summaryCost.getId(), parentId)).collect(Collectors.toList());
```

**2.筛选出List中的某个字段，形成新的集合**

```java
List<Long> deleteIdList = deleteList.stream().map(CollectArtificialCost::getId).collect(Collectors.toList());
```

**3.使用聚合函数，筛选出最大值**

```java
CostCollectProjects lastCollectProject = oldCollectProjectsList.stream().max(Comparator.comparing(CostCollectProjects::getCreateTime)).get();
```

**4.条件筛选出List中的某个实体**

```java
 CollectSummaryCost firstHeadItem = list.stream().filter(item -> com.alibaba.druid.util.StringUtils.equals(item.getSno(), "一")).findAny().orElse(null);
```

**5.筛选某个字段，形成string**

```java
String projectNames = submitCostProjectList.stream().map(CostCollectProjectsDto::getProjectName).collect(Collectors.joining(","));
```

**6.使用sort排序**
直接基于原list排序，无需重新赋值
```java
// 先根据reviewTime排序，再根据addtime排序，为null的排在后面
result.sort(Comparator
                    .comparing(RepReviewInfoDto::getReviewTime, Comparator.nullsLast(Comparator.naturalOrder()))  
                    .thenComparing(RepReviewInfoDto::getAddtime, Comparator.nullsLast(Comparator.naturalOrder())));
```

