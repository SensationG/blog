---
layout: post
title: 小程序 + vant UI开发笔记
date: 2020-07-27
Author: hhw
toc: true
comments: true
tags:  [vant-ui, JS]
pinned: true
---
小程序开发笔记+vant UI
记录开发时常用组件的使用方法 & <br>
开发时遇到的问题 <br>
文档：<br>
[vantUI](https://youzan.github.io/vant-weapp/#/intro) <br>
[小程序](https://developers.weixin.qq.com/miniprogram/dev/component/)


## 表单组件

### 1.修改属性

直接修改data的某个属性

```javascript
this.setData({
	code:'007'
})
123
```

修改data里的数组或对象的属性

```javascript
this.setData({
	'baseInfo.age':24
})
```

### 2.双向绑定

> 由于小程序只能双向绑定属性，无法双向绑定对象中的属性，所以只能自定义方法进行绑定。
>
> 以下是针对field输入框的绑定样例

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

> ”
>
> 以上只针对简单的field输入框进行绑定，如果是picker还需要修改
>
> “

**picker通用绑定方法 **（除picker日期外）

```js
// picker(除日期)通用绑定
onConfirm(event) {
  const name = event.currentTarget.dataset.name;
  const dataName = 'form.' + name;
  const value = event.detail.value;
  this.setData({
   // 绑定表单
 	 [dataName]: value.dictValue,
   // 回显
 	 [name]: value.dictLabel,
  })
  this.onClose();
},
```

```html
 <van-field
    required
    label="证件类型"
    placeholder="请选择证件类型"
    value="{{ credentialType }}"
    readonly
    data-name="showCredentialType"
    bindtap="onOpen"
    error-message="{{ credentialTypeMsg }}"
  />
  <!-- 证件类型选择弹出层 -->
  <van-popup
  show="{{ showCredentialType }}"
  position="bottom"
  custom-style="height: 30%;"
  >
    <van-picker 
      show-toolbar 
      data-name="credentialType"
      columns="{{ credentialTypeOption }}"
      value-key="dictLabel"
      title="请选择证件类型" 
      bind:confirm="onConfirm"
      bind:cancel="onClose" />
  </van-popup>
```

**picker日期绑定方法**

需要调用时间格式化方法和时间戳

```js
// picker日期通用绑定
onDate(e) {
  const name = e.currentTarget.dataset.name;
  const dataName = 'form.' + name;
  const value = e.detail;
  // 关闭弹框+时间格式化
  this.onClose();
  let res = this.formatterTime(value);
  this.setData({
    [name]: res,
    // 回显
    [dataName]: value,
  })
},
// 时间格式化
  formatterTime(time){
    //创建对象
    let date = new Date(time);
    //获取年份
    let y = date.getFullYear();
    //获取月份  返回0-11
    let m =date.getMonth()+1;
    // 获取日
    let d = date.getDate();
    //获取星期几  返回0-6   (0=星期天)
    let w = date.getDay();
    //星期几
    let ww = ' 星期'+'日一二三四五六'.charAt(date.getDay()) ;
    //时
    let h = date.getHours();
    //分
    let minute = date.getMinutes()
    //秒
    let s = date.getSeconds();
    //毫秒
    let sss = date.getMilliseconds() ;

    if(m<10){
      m = "0"+m;
    }
    if(d<10){
      d = "0"+d;
    }
    if(h<10){
      h = "0"+h;
    }
    if(minute<10){
      minute = "0"+minute;
    }
    if(s<10){
      s = "0"+s;
    }
    if(sss<10){
      sss = "00"+sss;
    }else if(sss<100){
      sss = "0"+sss;
    }
    return y+"-"+m+"-"+d;
  }
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

> 通常情况下，Picker搭配输入框Field以及弹出层Popup进行使用，以下是使用示例

```html
  <!-- ***通过输入框点击事件，显示Popup弹框，弹框内嵌Picker进行选择 -->
<van-field
    required
    label="证件类型"
    placeholder="请选择证件类型"
    value="{{ credentialType }}"
    readonly
    data-name="showCredentialType"
    bindtap="onOpen"
    error-message="{{ credentialTypeMsg }}"
  />

  <!-- 证件类型选择弹出层 show: 是否弹出 position: 弹出位置 -->
  <van-popup
  show="{{ showCredentialType }}"
  position="bottom"
  custom-style="height: 30%;"
  >
    	<!-- picker show-toolbar 显示确认按钮 bind:confirm 确认按钮事件 bind:cancel 取消按钮 				       事件 value-key显示遍历的内部属性值
			-->
  	<van-picker 
        show-toolbar 
        data-name="credentialType"
        columns="{{ credentialTypeOption }}"
        value-key="dictLabel"
        title="请选择证件类型" 
        bind:confirm="onConfirm"
        bind:cancel="onClose" 
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
  // 证件类型,这里实际上是从数据字典获取，这里是模拟数据。
  credentialTypeOption: [{dictLabel:'身份证', dictValue: 1},{dictLabel:'港澳通行证', dictValue: 2},   {dictLabel:' 台湾通行证', dictValue: 3},{dictLabel:'护照', dictValue: 4}],
  // 证件类型选择弹出层
  showCredentialType: false
},
// 通用弹出层弹出事件
onOpen(e) {
  const name = e.currentTarget.dataset.name;
  this.setData({
    [name]: true,
  })
},
// Picker 双向绑定
onCredentialTypeConfirm(event) {
   const name = event.currentTarget.dataset.name;
   const dataName = 'form.' + name;
   const value = event.detail.value;
  
   this.setData({
      // 绑定表单
      [dataName]: value.dictValue,
      // 回显
      [name]: value.dictLabel,
   })
   this.onClose();
},
// 所有弹出层关闭事件
onClose() {
  this.setData({
    showNation: false,
    showCredentialType: false,
    showBirthday: false,
    showLearn: false,
    showMarriageTime: false,
    showLifeStatus: false,
    textareaHidden: false
  })
},
```

### 5.van-Radio

由于官方文档只给出了块级元素的排列规则，如果我们想要把多个单选按钮排在同一行输入框表单中，就需要作出修改。

**原官方：**

<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20200730113738.png" alt="image-20200730112928644" style="zoom:50%;" />

**先给出目标效果图：**

<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20200730113731.png" alt="image-20200730113712052" style="zoom:50%;" />

**思路：**我们可以利用输入框 `van-field` 提供的button插槽将 `van-Radio` 置入输入框中，然后将原输入框的显示区域设为none，再调整Radio的位置即可。

**开始整改：**

- 首先将Radio嵌入field，设置 `van-field` 的 `use-button-slot` 属性为true
- 在要被嵌入的 `van-radio-group` 标签设置 `slot="button"`，即可完成嵌入

```html
<van-field
    label="性别"
    required
    use-button-slot   
    readonly
    class="f1"
    error-message="{{ sexMsg }}"
  >
  <van-radio-group data-name="sex" bind:change="onFormChange" value="{{ form.sex }}" 	            slot="button">
    <van-row>
      <van-col span="12" wx:for="{{ sexOption }}" wx:key="key">
        <van-radio 
          name="{{ item.dictValue }}"
        >
          {{ item.dictLabel }}
        </van-radio>
      </van-col>
    </van-row>
  </van-radio-group>
</van-field>
```

- 对样式进行调整

```css
/* 单选框样式 */
.f1 .van-field__input {
  display: none;
}
/* 插槽宽度调整 */
.f1 .van-field__button {
  width: 250px;
}
```

- JS绑定Radio即可

  

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

> 表单校验前使用`WxValidate.checkForm(this.data.form)`进行校验

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



## 路由导航

**1.导航组件navigator**

```html
<!-- Sample --> 
<navigator url="../form/form" >组件跳转测试</navigator>
```

URL适用于跳转小程序内的链接。[开发文档](https://developers.weixin.qq.com/miniprogram/dev/component/navigator.html)。可以通过设置open-type属性来切换跳转方式，默认是navigateBack。

![image-20200803172521241](https://raw.githubusercontent.com/SensationG/images/master/note/20200803193046.png)

如何跳转其他小程序？

修改target属性-->设置app-id属性为目标小程序的appid，path属性可以设置要打开目标小程序的页面路径--->extra-data属性可以用于传参。具体见开发文档。

**2.wx.navigateTo**

```js
wx.navigateTo({
	url: '../form/form',
  success(res) {
     res.eventChannel.emit('acceptDataFromOpenerPage', { data: 'test' })
  }
})
```

保留当前页面，跳转到应用内的某个页面。但是不能跳到 tabbar 页面。url后可以携带参数。

如何向被打开页面传送参数？使用success回调函数，如上

被打开页面接收：

```js
// 监听acceptDataFromOpenerPage事件，获取上一页面通过eventChannel传送到当前页面的数据
const eventChannel = this.getOpenerEventChannel()
eventChannel.on('acceptDataFromOpenerPage', function(data) {
  console.log(data)
})
```

** 这里可能会出现接收不到的情况，这时需要使用Promise函数等待传参过来后再执行：

```js
 new Promise((resolve,reject) => {
   const eventChannel = this.getOpenerEventChannel()
   eventChannel.on('enrollPerson', function(data) {
     console.log(data)
     resolve()
   })
 }).then(res => {

 })
```



使用 [wx.navigateBack](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateBack.html) 可以返回到原页面。小程序中页面栈最多十层。

**3.wx.navigateBack**

关闭当前页面，返回上一页面或多级页面。可通过 [getCurrentPages](https://developers.weixin.qq.com/miniprogram/dev/reference/api/getCurrentPages.html) 获取当前的页面栈，决定需要返回几层。

[API](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateBack.html)

**4.wx.redirectTo**

关闭当前页面，跳转到应用内的某个页面。但是不允许跳转到 tabbar 页面。

[传值](https://developers.weixin.qq.com/community/develop/doc/000c2a15418d608d1f6ace42156c00)

**5.wx.reLaunch**

关闭所有页面，打开到应用内的某个页面

**6.wx.switchTab**

跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面

## 小程序发布体验版

注：由于小程序开发版仅供开发人员自己测试（测试号(APPID)仅供开发人员自己的微信账号测试），倘若需要将开发中的小程序提供给其他人测试使用，那么需要<u>发布小程序体验版</u>。

**步骤一：去小程序后台填写相关信息**

[小程序后台链接](https://mp.weixin.qq.com/)

登入完成后，点击首页--->点击完善按钮

![image-20200831092932231](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20200831105004.png)

*由于图中信息已经填写完成，所以是查看详情按钮，若未填写完成，则是完善信息按钮。

**步骤二：完善基本信息**

因为是发布体验版，所以这里不需要审核，基本信息如实填写即可

**步骤三：添加体验成员**

点击成员管理，将需要使用体验版的微信号添加到体验成员列表

![image-20200831093428817](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20200831105010.png)

直接搜索微信号添加即可

**步骤四：上传代码**

打开微信开发者工具，点击上传按钮

![image-20200831093543513](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20200831105014.png)

**步骤五：到小程序后台将此次上传版本设置为体验版**

![image-20200831093724719](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20200831105019.png)

设置完成后，扫描体验版的二维码即可访问

<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20200831105023.png" alt="image-20200831093755541" style="zoom: 33%;" />

## 自定义音频播放组件

> 由于官方提供的音频组件无法提供进度条拖动功能，所以我们需要自定义一个音频播放组件
>
> 使用官方api: wx.createInnerAudioContext()  创建 InnerAudioContext 实例从而控制音频播放，前端播放UI需要完全自定义

已知bug：1、小程序开发组件下进度条拖动问题，但真机实测无该问题。

​				  2、安卓机情况下，要播放开始才能获取音频总时长信息

​				  3、多个组件不能同时放在同一页面，因为全局只能存在一个wx.createInnerAudioContext实例，如果						需要在同一个页面放多个组件，那么目前的解决方法是使用wx:if控制组件的生命，使同一个page一						次只能有一个组件出现

**使用方法**

> 这里以创建自定义播放组件为例

**创建InnerAudioContext实例**

在Page外创建实例：

```js
const innerAudioContext = wx.createInnerAudioContext()
```

**设置监听器，基础信息**

在onload或cteated中创建监听器，只需要执行一次即可使用

注：实测在自定义组件的父传子参数prop的observer(数据发送变化时调用)函数中调用也可以使用

```js
properties: {
    name: {
      type: String,
      value: '无信息'
    },
    src: {
      type: String,
      observer: function(data) {   
        this.initAudio(data)
      }
    },
    author: {
      type: String
    }
  },
  data: {
    name: undefined,
    // 是否播放
    play: false,
    // 进度条百分比
    audioPos: undefined,
    // 当前播放时常
    audioCurrent: '00:00',
  },
// 以在observer函数中调用为例
methods: {
  initAudio(data) {
    this.initParam()
    // 设置音频链接
    innerAudioContext.src = data
    // 在onCanplay里获取并设置音频时长和播放进度
    innerAudioContext.onCanplay(() => {
      // 这里必须初始化，否则获取不到音频时长
      innerAudioContext.duration
      // 必须使用定时器，否则获取不到音频时常
      setTimeout(() => {
        this.setData({
          audioDuration: this.format(innerAudioContext.duration),
        });
      }, 500);
    });
    // 播放进度更新
    innerAudioContext.onTimeUpdate(() => {
      this.setData({
        audioPos: innerAudioContext.currentTime / innerAudioContext.duration * 100,
        audioCurrent: this.format(innerAudioContext.currentTime)
      })
    })
    // 录音播放暂停
    innerAudioContext.onPause(() => {
      this.setData({
        play: false
      })
    })
    // 音频播放完成
    innerAudioContext.onEnded(() => {
      this.setData({
        play: false
      })
    })
    innerAudioContext.onError((res) => {
      // Toast.fail('音频加载失败')
      console.log('音频加载失败')
    })
  },
}
```

**界面样式**

<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20200904110854.png" alt="image-20200904110839141" style="zoom:50%;" />

基于小程序原生滑动组件和vant-ui的按钮/icon制作

包含功能：

- 播放/暂停
- 显示名称，作者
- 当前播放时长（由于音频播放总时长在安卓机上存在问题，所以暂未显示）
- 播放进度条，可拖拽调整进度

**注意点**

- 在组件从节点树被移除时，需要先停止播放，在销毁实例，否则可能还会继续播放音乐（疑似实例没有被真正销毁）

```js
// 组件实例从页面节点树移除
detached() {
  this.initParam() //初始化参数（可选）
  innerAudioContext.stop()
  innerAudioContext.destroy()
},
```

- 进度条的使用必须切换成百分比赋值，监听器获取的时长是毫秒，也必须化成分钟

```js
// 拖动进度条，到指定位置
sliderChange(e) {
  const position = e.detail.value;
  // 换算百分比
  const currentTime = position / 100 * innerAudioContext.duration;
  innerAudioContext.seek(currentTime);
  this.setData({
    audioPos: position,
    audioCurrent: this.format(currentTime)
  })
},
// 时间格式化 
format(t) {
  let time = Math.floor(t / 60) >= 10 ? Math.floor(t / 60) : '0' + Math.floor(t / 60);
  t = time + ':' + ((t % 60) / 100).toFixed(2).slice(-2);
  return t;
}
```

完整代码已上传GitHub: https://github.com/SensationG/wxminiprogram-component

## 获得用户昵称和头像

**方式一 **

使用官方Open-data标签，不需要用户授权就可以直接获取头像和昵称

```html
 <open-data type="userAvatarUrl"></open-data>  //获取用户头像直接显示在小程序中
 <open-data type="userNickName" lang="zh_CN"></open-data>  //获取用户昵称直接显示在小程序中
```

**方式二**

使用button配合open-type，此方式会弹出用户授权提示框，需要用户手动确认

```html
<button open-type='getUserInfo' lang="zh_CN" bindgetuserinfo="onGotUserInfo"></button>
```

```js
onGotUserInfo: function (e) {
  console.log("nickname=" + e.detail.userInfo.nickName);
}
```

## 轮播图组件

使用官方的轮播图组件swiper + swiper-item [文档](https://developers.weixin.qq.com/miniprogram/dev/component/swiper.html)

使用示例：

```html
<!-- 轮播图 -->
<view class="swiper_pic">
  <swiper indicator-dots="{{indicatorDots}}"
          autoplay="{{autoplay}}" interval="{{interval}}" duration="{{duration}}" style="height: 200px;">
    <block wx:for="{{background}}" wx:for-item="item" wx:key="*this">
      <swiper-item>
        <image src="{{ item }}" mode="scaleToFill" class="pic"/>
      </swiper-item>
    </block>
  </swiper>
</view>
```

js参数

```js
// 图片链接
background: ['https://raw.githubusercontent.com/SensationG/images/master/note/20200908170304.png', 'https://raw.githubusercontent.com/SensationG/images/master/note/20200908170734.png', 'https://raw.githubusercontent.com/SensationG/images/master/note/20200908165205.png'],
 // 是否显示面板指示点
 indicatorDots: true,
 // 滑动方向是否为纵向
 vertical: false,
 // 自动播放
 autoplay: true,
 // 自动切换时间间隔
 interval: 4000,
 // 滑动动画时长
 duration: 500
```









