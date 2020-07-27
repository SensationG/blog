---
layout: post
title: 小程序开发笔记+vant UI
date: 2020-07-27
Author: hhw
categories: 
tags:  [vant-ui, js]
comments: false
---
小程序开发笔记+vant UI
记录开发时常用组件的使用方法 &
开发时遇到的问题

## 表单组件

### 1.修改属性

#### 直接修改data的某个属性

```javascript
this.setData({
	code:'007'
})
123
```

#### 修改data里的数组或对象的属性

```javascript
this.setData({
	'baseInfo.age':24
})
```

### 2.双向绑定

> 由于小程序只能双向绑定属性，无法双向绑定对象中的属性，所以只能自定义方法进行绑定。

```js
data: {
  loginForm: {
    username: "",
    password: "",
    rememberMe: false,
    code: "",
    uuid: ""
  }
},
// 双向绑定事件，组件内容改变时调用
onLoginFormChange(e){
    // 获取组件上的自定义属性名 set-xxx='xxx' ,这里是set-name 故使用.name获取
    const name = e.currentTarget.dataset.name;
    // 设置要绑定的对象的属性，这里的loginForm.是对象名，根据实际需求变更。
    const dataName = 'loginForm.' + name;
  	// 获取组件的输入值
    const value = e.detail;
  	// 设置属性
    this.setData({
      [dataName]: value
    })
  }
```

实际使用时，只需变更对象名即可，即 loginForm所在位置。

wxml：

```html
<!-- bind:change 小程序原生，当input内容发生改变时调用 -->
<!-- data-name 自定义属性名，可以是data-xxx，可以在bind:change中的形参e获取，方便设置值 -->
<van-cell-group>
  <van-field         
    bind:change="onLoginFormChange"
    data-name="username"
    required
    clearable
    label="用户名"
    placeholder="请输入用户名"
    auto-focus
  />

  <van-field
    bind:change="onLoginFormChange"
    data-name="password"
    type="password"
    label="密码"
    placeholder="请输入密码"
    required
    border="{{ false }}"
  />

  <van-button round type="info" bind:click="submit" >提交</van-button>
</van-cell-group>
```

### 3.Toast提示

报错问题：

官方给的路径:
```
import Toast from 'path/to/@vant/weapp/dist/toast/toast';
```

报错` not defined`    修改为
```
import Toast from '@vant/weapp/toast/toast';
```
可以正常使用了
记得在wxml添加组件才能显示，id不要变更：

```html
<van-toast id="van-toast" />
```

### 4.van-picker

小程序使用的是 picker组件的`range`和`range-key `属性进行选项展示，而vant-ui是使用`van-picker`组件的

 `columns` 和 `value-key` 属性进行选择器的选项展示。

> #### 通常情况下，Picker搭配输入框Field以及弹出层Popup进行使用，以下是使用示例

```html
	<!-- ***通过输入框点击事件，显示Popup弹框，弹框内嵌Picker进行选择 -->
	<view bindtap="chooseCredentialType" >
    <van-field
      required
      label="证件类型"
      placeholder="请选择证件类型"
      value="{{ credentialName }}"
      readonly
    />
  </view>

  <!-- 证件类型选择弹出层 
		show: 是否弹出
		position: 弹出位置
	-->
  <van-popup
  show="{{ show }}"
  position="bottom"
  custom-style="height: 30%;"
  >
    <!-- picker
			show-toolbar: 显示确认按钮
			bind:confirm: 确认按钮事件
			bind:cancel: 取消按钮事件
		-->
  	<van-picker 
        show-toolbar 
        columns="{{ credentialType }}"
        value-key="dictLabel"
        title="请选择证件类型" 
        bind:confirm="onCredentialTypeConfirm"
        bind:cancel="onCredentialTypeClose" 
     />
  </van-popup>
```

-  field下的 `readonly` ，让输入框field为只读状态，只能通过js方法进行修改。
-  picker下的 `columns` 与 `value-key`，实现从对象数组读取选项。<u>columns绑定对象数组，value-key直接绑定columns中的属性。</u>

- 当确认按钮点击时，`bind:confirm` 绑定的事件将会执行，因此从这里对目标数据进行绑定。

```js
data: {
  // 表单
  form: {
    name: '',
    mobile: '',
    credentialType: '',
    credentialNumber: '',
  },
  // 选中后的证件类型回显
  credentialName: '',
  // 证件类型,这里实际上是从数据字典获取，这里是模拟数据。
  credentialType: [{dictLabel:'身份证', dictValue: 1},{dictLabel:'港澳通行证', dictValue: 2},   {dictLabel:' 台湾通行证', dictValue: 3},{dictLabel:'护照', dictValue: 4}],
  // 证件类型选择弹出层
  show: false
},
// 点击输入框事件
chooseCredentialType() {
    this.setData({
      show:true
    })
},
// Picker 确认按钮事件
onCredentialTypeConfirm(event) {
   this.setData({
      show: false,
      'form.credentialType': event.detail.value.dictValue,
      credentialName: event.detail.value.dictLabel
   })
},
// Picker 取消按钮事件
onCredentialTypeClose() {
   this.setData({
      show:false
   })
}
```

## 表单校验

使用官方社区开发的WxValidate进行表单校验

> WxValidate插件是参考 jQuery Validate 封装的，为小程序表单提供了一套常用的验证规则，包括手机号码、电子邮件验证等等，同时提供了添加自定义校验方法，让表单验证变得更简单。

### 引入

[下载地址](https://github.com/skyvow/wx-extend)，下载完成后，将WxValidate.js引入utils目录下：

<img src="https://raw.githubusercontent.com/SensationG/images/master/note/20200727155141.png" alt="image-20200727115728041" style="zoom:50%;" />

之后在需要使用的页面使用import进行引入即可，注意路径准确性，示例：

```js
// 引入表单校验插件
import WxValidate from '../../utils/wx-validate'
```

### 绑定校验数据

> 需要注意的是，需要验证的input框内必须使用value属性绑定值，绑定到该input框所在的form表单下的对应属性。即双向绑定。

示例：在js编写表单对象

```js
data: {
  // 表单
  form: {
    name: '',
    mobile: '',
  }
},
```

在wxml编写表单组件，并双向绑定js中-表单对象form内的-对应属性。

> 由于小程序无法双向绑定对象中的属性，所以使用自定义双向绑定方法进行绑定，这里不使用value
>
> ```js
> // 双向绑定事件，组件内容改变时调用
> onLoginFormChange(e){
>     // 获取组件上的自定义属性名 set-xxx='xxx' ,这里是set-name 故使用.name获取
>     const name = e.currentTarget.dataset.name;
>     // 设置要绑定的对象的属性，这里的loginForm.是对象名，根据实际需求变更。
>     const dataName = 'loginForm.' + name;
>   	// 获取组件的输入值
>     const value = e.detail;
>   	// 设置属性
>     this.setData({
>       [dataName]: value
>     })
>   }
> ```

```html
<van-cell-group>

  <van-field
    required
    clearable
    label="姓名"
    placeholder="请输入姓名"
    data-name="name"
    bind:change="onFormChange"
    auto-focus
  />

   <van-field
    required
    clearable
    label="电话号码"
    placeholder="请输入电话号码"
    data-name="mobile"
    bind:change="onFormChange"
  />
  
  <van-field
    required
    clearable
    label="证件号"
    placeholder="请输入证件号"
    data-name="credentialNumber"
    bind:change="onFormChange"
  />

  <van-button type="primary" block bind:click="submit">提交</van-button>

</van-cell-group>

<!-- 校验规则不通过的提示 -->
<van-toast id="van-toast" style="width: 300px" />
```

### 校验规则使用

#### 内置校验规则

| 序号 | 规则                   | 描述                                                    |
| ---- | ---------------------- | ------------------------------------------------------- |
| 1    | `required: true`       | 这是必填字段。                                          |
| 2    | `email: true`          | 请输入有效的电子邮件地址。                              |
| 3    | `tel: true`            | 请输入11位的手机号码。                                  |
| 4    | `url: true`            | 请输入有效的网址。                                      |
| 5    | `date: true`           | 请输入有效的日期。                                      |
| 6    | `dateISO: true`        | 请输入有效的日期（ISO），例如：2009-06-23，1998/01/22。 |
| 7    | `number: true`         | 请输入有效的数字。                                      |
| 8    | `digits: true`         | 只能输入数字。                                          |
| 9    | `idcard: true`         | 请输入18位的有效身份证。                                |
| 10   | `equalTo: 'field'`     | 输入值必须和 field 相同。                               |
| 11   | `contains: 'ABC'`      | 输入值必须包含 ABC。                                    |
| 12   | `minlength: 5`         | 最少要输入 5 个字符。                                   |
| 13   | `maxlength: 10`        | 最多可以输入 10 个字符。                                |
| 14   | `rangelength: [5, 10]` | 请输入长度在 5 到 10 之间的字符。                       |
| 15   | `min: 5`               | 请输入不小于 5 的数值。                                 |
| 16   | `max: 10`              | 请输入不大于 10 的数值。                                |
| 17   | `range: [5, 10]`       | 请输入范围在 5 到 10 之间的数值。                       |

> 此处需要注意的是一定要在js文件中onLoad验证规则，否则编译会报checkform is not a function 

```js
// 校验方法必须在onLoad中执行初始化
onLoad() {
	this.initValidate();
},
// 表单校验方法，方法名可自定义 -使用内置校验规则
initValidate() {
  const rules = {
    name: {
      required: true,
      minlength:2
    },
    mobile:{
      required:true,
      tel:true
    }
	}
  const messages = {
    name: {
      required: '请填写姓名',
      minlength:'请输入正确的姓名'
    },
    mobile:{
     required:'请填写手机号',
     tel:'请填写正确的手机号'
    }
  }
  // 创建实例对象
	this.WxValidate = new WxValidate(rules, messages)
}
```

```js
// 表单提交
submit() {
   console.log(this.data.form)
    // 调用校验方法，传入表单form对象
   if (!this.WxValidate.checkForm(this.data.form)) {
     const error = this.WxValidate.errorList[0];
     this.showModal(error);
     return false;
   }
   Toast('提交成功');
}
// 搭配toast显示表单错误信息
showModal(error) {
  Toast({
    message: error.msg,
  });
}
```

#### 自定义校验规则

addMethod(name, method, message) - 添加自定义校验。包含三个参数，name：添加的方法的名字；method：校验方法，在这里添加自定义校验规则，返回true/false；message：自定义错误提示；

> 使用addMethod添加自定义校验规则后，使用方法大部分都与内置校验规则相同

```js
 // 表单校验
initValidate() {
  const rules = {
    name: {
      required: true,
      minlength:2
    },
    mobile:{
      required:true,
      tel:true
    },
    // 使用自定义校验
    credentialNumber: {
      required: true,
      // 自定义校验方法设为true
      credentialNumber: true,
    } 
  }
  const messages = {
    name: {
      required: '请填写姓名',
      minlength:'请输入正确的姓名'
    },
    mobile:{
      required:'请填写手机号',
      tel:'请填写正确的手机号'
    },
    // 使用自定义校验
    credentialNumber: {
      required: '请输入证件号',
      credentialNumber: '请填写正确的证件号',
    }
  }
  // 创建实例对象
  this.WxValidate = new WxValidate(rules, messages)
  
  // 自定义验证规则
  this.WxValidate.addMethod('credentialNumber', (value, param) => {
    // 添加正则
    const IDCardReg =  /^\d{6}(18|19|20)?\d{2}(0[1-9]|1[0-2])(([0-2][1-9])|10|20|30|31)\d{3}(\d|X|x)$/;
    if (IDCardReg.test(value)) {
      // 校验通过
      return true;
    } else {
      return false;
    }
  })
}
```


