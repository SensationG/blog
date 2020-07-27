---
layout: post
title: ElementUI使用问题归纳
date: 2020-04-10
Author: hhw
toc: true
comments: true
tags:  [ElementUI,Vue,JS]
---

## 1.el-form 表单一行多个输入框
我们使用饿了么表单的时候，通常会使用 `<el-form-item>`标签包裹输入框，如下：

```
<el-form-item label="参加活动情况:">
    <el-input value="等待活动模块完成"></el-input>
</el-form-item>
```
默认情况下，都是一行一个输入框，但是如何才能一行多个输入框？，我们可以使用属性
`inline	行内表单模式	boolean	—	false` 使得表单输入框为行内模式：<br>
```
<el-form :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
```
并且使用`label-width="100px"`来控制表单输入框的宽度从而控制一行输入框的个数
如下
 
```
<el-form-item label="参加活动情况:" label-width="100px">
    <el-input></el-input>
</el-form-item>
```
## 2.el-form 重置表单失效并且重置后无法输入的问题（RuoYi下自带重置方法）
失效的原因是没有加prop属性，需要在el-form-item标签配置prop属性，并且需要在date区域列出所有表单v-model的属性并且赋值为undefined；如下：<br>
*data中定义：*
```js
 // 查询参数
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    trueName: undefined,
    sex: undefined,
  },
```
*form查询表单：*

```html
<el-form :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
  <!-- 注意配置prop -->
  <el-form-item label="姓名" prop="trueName">
    <el-input
      v-model="queryParams.trueName"
      placeholder="请输入姓名"
      clearable
      size="small"
      @keyup.enter.native="handleQuery"
    />
  </el-form-item>
  <el-form-item label="性别" prop="sex">
    <el-select v-model="queryParams.sex" placeholder="全部" size="small">
      <el-option
        v-for="(dict,key) in sexOptions"
        :label="dict.dictLabel"
        :value="dict.dictValue"
        :key="key"
      />
    </el-select>
  <el-form-item>
    <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
    <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
  </el-form-item>
</el-form>
```
*重置表单方法：*

```js
 /** 重置按钮操作 */
resetQuery() {
  // ruoyi封装了resetForm方法
  this.resetForm("queryForm");
  // 重置完查询表单重新执行查询动作
  this.handleQuery();
},
```
## 3.el-form 的 :model 属性的作用
表单中的输入框使用v-model绑定后，el-form有无model没有任何影响，可以删除不写
目前，el-form的model主要用于表单验证，也就是配合el-form的rules和el-form-item的prop来使用；
主要是配合表单验证
## 4.表单的clearable属性/回车方法
`clearable`作用在输入框上，作用：输入框是否显示清空按钮，
`@keyup.enter.native="handleQuery"` 激活回车方法，如下：
 
```html
<el-form-item label="姓名" prop="trueName">
    <el-input
      v-model="queryParams.trueName"
      placeholder="请输入姓名"
      clearable
      @keyup.enter.native="handleQuery"
    />
</el-form-item>
```

