---
layout: post
title: JWT与Session对比
date: 2020-07-25
Author: hhw
toc: true
comments: true
tags:  [java]
pinned: true
---
 认证授权2种方式：Session-Cookie 与 JWT，下面我们就针对这两种方案就行阐述。<br>
 包括两种方式的工作原理以及认证流程。


#### 基于Session-Cookie

##### 工作原理

当Client输入账号密码后，Server会产生一笔Session记录，同时把SessionID传送回给Client端加密保存在Cookie中（只有Server能解开）。之后再次请求Server，Client就会携带Cookie，其中包含有SessionID，Server根据该SessionID来识别身份，获取到当前用户的Session。

当想要清除的话，除了等待Session过期，还可以使用Server注销当前Session，另外也可以清除Client的Cookie。

###### 与Cookie的差别

基于Cookie，需要依赖Cookie来保存SessionID，但是实际的Session还是保存在Server端。主要是一个承载SessionID的作用。

##### 优势

- 相比JWT，最大的优势就在于可以主动清除session了，JWT只能设置过期时间
- session保存在服务器端，相对较为安全

##### 劣势

- cookie + Sesssion 跨域存在问题

  如果是分布式部署，需要做多机共享session机制。

  

##### sessionStorage、localstorage

> session: 主要存放在服务器端，相对安全

> cookie: 可设置有效时间，默认是关闭浏览器后失效，主要存放在客户端，并且不是很安全，可存储大小约为4kb

> sessionStorage: 仅在当前会话下有效，关闭页面或浏览器后被清除

> localstorage: 除非被清除，否则永久保存


### 基于JWT

##### JWT是什么

> *原名 ( JSON Web Tokens )，基本上就是帶時效的 Token。*

JWT主要分为三段，分别是头部，有效载荷和签名。

```
Header.Payload.Signature
```

分为三段，之间使用 . 分隔，每一段都使用Base64Url去编码，中间的Payload有时会加密

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoiemhhbmdzYW4ifQ.ec7IVPU-ePtbdkb85IRnK4t4nUVvF2bBf8fGhJmEwSs
```

**Base64Url**

与 BASE64 差在最后的 「 **=** 」 号会去掉，「 **- ` /**」符号会换成底线「 **_** 」， 「**+**」会换成 dash「 **-** 」。

##### Header

JWT 的 Header 通常包含两个字段，分别是：typ(type) 和 alg(algorithm)。

- typ：token的类型，这里固定为 JWT
- alg：加密方式，例如：HMAC SHA256 或者 RSA

一个简单的例子：

```json
{
	"alg": "HS256",
	"typ": "JWT"
}
```

使用Base64Url编码后：

```
>>> base64UrlEncode({"alg":"HS256","typ":"JWT"})
最终结果是： 'eyJhbGciOiAiSFMyNTYiLCAidHlwIjogIkpXVCJ9' 
```

##### Payload

存放需要传递的信息，例如用户名，邮箱，发布人，过期时间等信息。

官方提供的属性有：

```
iss: 发行人
exp: 到期日
sub: 主题
aud: 收件人
nbf: 不接受早于…日期/时间
iat: 发行时间
jti: 唯一辨识符，JWT 只能使用一次
```

举例：

```json
{
	"userName":"lisi"
}
```

转Base64Url后

```
eyJ1c2VyX2lkIjoiemhhbmdzYW4ifQ
```

##### Signature

Signature对我们前面的 Header 和 Payload 部分进行签名，保证 Token 在传输的过程中没有被篡改或者损坏。

完整算法为：

```
Signature = HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
secret)
```

secret可以打上自己想要的。

**最后把这三段拼在一起，就是JWT**



##### 工作原理

1.首先，前端通过Web表单将自己的用户名和密码发送到后端的接口。

2.后端核对用户名和密码成功后，将用户的id等其他信息作为 JWT Payload（负载），将其与头部分别进行Base64编码拼接后签名，形成一个JWT。形成的JWT就是一个形同lll.zzz.xxx的字符串。

3.后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在localStorage或sessionStorage上，退出登录时前端删除保存的JWT即可。

4.前端在每次请求时将JWT放入HTTP Header中的Authorization位。(解决XSS和XSRF问题)

5.后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确；检查Token是否过期；检查Token的接收方是否是自己（可选）。

6.验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。




##### 优劣

**RESTful API服务**

现代应用程序的常见模式是从RESTful API查询使用JSON数据。目前大多数应用程序都有RESTful API供其他开发人员或应用程序使用。由API提供的数据具有几个明显的优点，其中之一就是这些数据可以被多个应用程序使用。在这种情况下，传统的使用session和Cookie的方法在用户认证方面效果不佳，因为它们将状态引入到应用程序中。

RESTful API的原则之一是它应该是无状态的，这意味着当发出请求时，总会返回带有参数的响应，不会产生附加影响。用户的认证状态引入这种附加影响，这破坏了这一原则。保持API无状态，不产生附加影响，意味着维护和调试变得更加容易。

另一个挑战是，由一个服务器提供API，而实际应用程序从另一个服务器调用它的模式是很常见的。为了实现这一点，我们需要启用跨域资源共享（CORS）。Cookie只能用于其发起的域，相对于应用程序，对不同域的API来说，帮助不大。在这种情况下使用JWT进行身份验证可以确保RESTful API是无状态的，你也不用担心API或应用程序由谁提供服务。

**性能**

- JWT大小远超过SessionID，因此在HTTP请求中，需要更多开销；而对于Session，每个请求需要查找和反序列化Session
- JWT可能需要访问后端数据库来验证信息，而Session不需要。但可以在登录验证成功后将用户信息存入Redis，从而可以进行快速验证。

[参考1](https://juejin.im/post/5a437441f265da43294e54c3)<br>
[参考2](https://medium.com/@jedy05097952/%E6%B7%BA%E8%AB%87-session-%E8%88%87-jwt-%E5%B7%AE%E7%95%B0-8d00b2396115)
