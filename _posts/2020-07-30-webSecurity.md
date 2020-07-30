---
layout: post
title: XSS、CSRF漏洞介绍
date: 2020-07-30
Author: hhw
toc: true
comments: true
tags:  [JS,前端,安全]
---

常见的Web攻击手段：

- XSS攻击
- CSRF攻击
- SQL注入攻击
- 文件上传漏洞
- DDoS攻击
- 其他攻击手段

本文主要介绍前两种，第三种[SQL注入](https://sensationg.github.io/blog/SQLInjectionAttack/)已经在另外一篇提及。

## 一、XSS漏洞

XSS全称 “跨站脚本攻击”，是注入攻击的一种。原理是恶意web用户将代码植入到提供给其他用户的页面中，通过在网页中嵌入恶意脚本，当用户打开网页时，恶意脚本开始在用户浏览器上执行。XSS 跨站脚本攻击的危害：窃取 Cookie、放蠕虫、网站钓鱼。

#### XSS攻击分类

XSS 跨站脚本攻击主要可以分为：存储型 XSS、反射型 XSS、Dom 型 XSS。 

***存储型 XSS：***

也叫持久型XSS，主要将XSS代码提交存储在服务器端（数据库，内存，文件系统等），下次请求目标页面时不用再提交XSS代码。当目标用户访问该页面获取数据时，XSS代码会从服务器解析之后加载出来，返回到浏览器做正常的HTML和JS解析执行，XSS攻击就发生了。存储型 XSS 一般出现在网站留言、评论、博客日志等交互处，恶意脚本存储到客户端或者服务端的数据库中。

***反射型 XSS：***

一般是攻击者通过特定手法（如电子邮件），诱使用户去访问一个包含恶意代码的 URL，当受害者点击这些专门设计的链接的时候，恶意代码会直接在受害者主机上的浏览器执行。反射型XSS通常出现在网站的搜索栏、用户登录口等地方，常用来窃取客户端 Cookies 或进行钓鱼欺骗。

***Dom型 XSS：***

基于 DOM 的 XSS 攻击是指通过恶意脚本修改页面的 DOM 结构，是纯粹发生在客户端的攻击。DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞。

#### 漏洞案例

以反射型xss为例，攻击者通过电子邮件等方式将包含注入脚本的恶意链接发送给受害者，当受害者点击该链接的时候攻击发生，常见的反射型xss攻击步骤如下：

1. 攻击者在URL后面的参数中加入恶意攻击代码
2. 当用户打开带有恶意代码的URL时，网站将会解析执行这些恶意代码
3. 攻击者通过恶意代码来窃取用户数据并发送到攻击者网站然后冒充用户等等恶意行为

最常见的就是：恶意链接，案例如下

```html
<input type="text" value="<%= getParameter("keyword") %>">
<button>搜索</button>
<div>
  您搜索的关键词是：<%= getParameter("keyword") %>
</div>
```

当链接经过恶意修改后：`http://xxx/search?keyword="><script>alert('XSS');</script>`

点开链接，会发现中招了，浏览器会解析并执行这些恶意代码，最终变成了：

```html
<input type="text" value=""><script>alert('XSS');</script>">
<button>搜索</button>
<div>
  您搜索的关键词是："><script>alert('XSS');</script>
</div>
```

这时 input的value属性被注入了，下面的div也被注入了，将弹框2次。原因是浏览器把用户输入当成了脚本来执行。此时漏洞就可能会被植入很多恶意脚本，比如通过 javascript 代码来获取 cookies 信息然后再异步发送请求到指定服务器，这样就可以获取到用户登录凭证，用户可以通过登录凭证，绕过登录，对系统加以攻击。

#### 编程要求

通过以上示例可知， XSS 漏洞其核心就是利用脚本进行注入，因此我们要求不信赖用户输入，严格限制用户的输入，同时对` “>”、“<”、“’”、“"” `等特殊字符进行转义。同时后端也需要对提交的数据进行过滤。当然具体情况还是需要具体分析。



## 二、CSRF漏洞

CSRF(Cross-site request forgery)跨站请求伪造：攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击者网站发送跨站请求，利用受害者在攻击网站已经获取登录凭证，绕过后台用户登录验证，达到冒充用户对被共计的网站执行某项操作的目的。

#### 攻击流程

**攻击流程：** 

1、 受害者登录 A 网站，并保留登录凭证（Cookie）

2、 攻击者引诱受害者访问 B 网站

3、 B 网站向 A 网站发送一个请求：A 网站/role/save.do，浏览器默认会携带 A 网站的 Cookie

4、 A 网站接收到请求后，对请求进行验证，并确认是受害者的凭证，误以为是受害者自己发送的请求

5、 A 网站以受害者的名义执行了 A 网站/role/save.do

6、 攻击完成，攻击者在受害者毫不知情的情况下，冒充受害者，让 A 网站执行了自己定义的操作

![image-20200730224746487](https://raw.githubusercontent.com/SensationG/images/master/note/20200730224748.png)

#### 防御

防御 CSRF 攻击在应用层面主要有三种策略：

**验证** **HTTP Referer** **字段：**

根据 HTTP 协议，在 HTTP 头中有一个字段叫 Referer，它记录了该 HTTP 请求的来源地

址。在通常情况下，访问一个安全受限页面的请求必须来自于同一个网站。如果 Referer 是其

他网站的话，就有可能是 CSRF 攻击，则拒绝该请求。

**在请求地址中添加** **token** **并验证：**

CSRF 攻击之所以能够成功，是因为攻击者可以伪造用户的请求，该请求中所有的用户

验证信息都存在于 Cookie 中，因此攻击者可以在不知道这些验证信息的情况下直接利用用户

自己的 Cookie 来通过安全验证。由此可知，抵御 CSRF 攻击的关键在于：在请求中放入攻击

者所不能伪造的信息，并且该信息不存在于 Cookie 之中。鉴于此，系统开发者可以在 HTTP

请求中以参数的形式加入一个随机产生的 token，并在服务器端建立一个拦截器来验证这个

token，如果请求中没有 token 或者 token 内容不正确，则认为可能是 CSRF 攻击而拒绝该请求。

**在** **HTTP** **头中自定义属性并验证：**

自定义属性的方法也是使用 token 并进行验证，和前一种方法不同的是，这里并不是把

token 以参数的形式置于 HTTP 请求之中，而是把它放到 HTTP 头中自定义的属性里。通过

XMLHttpRequest 这个类，可以一次性给所有该类请求加上 csrftoken 这个 HTTP 头属性，并 

把 token 值放入其中。这样解决了前一种方法在请求中加入 token 的不便，同时，通过这个类

请求的地址不会被记录到浏览器的地址栏，也不用担心 token 会通过 Referer 泄露到其他网站。



参考文章：

[前端安全系列（一）：如何防止XSS攻击？](https://tech.meituan.com/2018/09/27/fe-security.html)

[web安全之XSS攻击原理及防范](https://www.cnblogs.com/tugenhua0707/p/10909284.html)

[常见web攻击方法及防御手段总结](https://blog.csdn.net/qappleh/article/details/80485197)
