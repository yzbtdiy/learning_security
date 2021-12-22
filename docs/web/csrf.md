# CSRF 

## CSRF 漏洞原理

CSRF(CrossSiteRequestForgery), 中文是跨站点请求伪造. CSRF攻击者在用户已经登录目标网站之后, 诱使用户访问一个攻击页面, 利用目标网站对用户的信任, 以用户身份在攻击页面对目标网站发起伪造用户操作的请求, 比如以你的名义发送邮件, 发消息, 盗取你的账号, 添加系统管理员, 甚至于购买商品, 虚拟货币转账等, 达到攻击目的.

CSRF 和 XSS 最大的区别就在于, CSRF 并没有盗取 cookie 而是直接利用. 

XSS 可以通过 `document.cookies` 获取 cookie .

### 漏洞场景

1. 用户打开浏览器, 访问受信任的网站, 但是该网站存在 CSRF 漏洞, 用户输入账号密码正常登录

2. 在用户信息通过验证后, 该网站产生了cookie信息并返回给浏览器, 用户登录成功, 可以正常操作

3. 用户为退出登录的情况下, 在同一浏览器中, 打开新的 Tab 访问了某个恶意网站

4. 恶意网站会返回一些有攻击性的代码, 在用户不知情的情况下利用用户的 cookie 信息对原网站执行某些操作

## CSRF 漏洞挖掘

CSRF 漏洞挖掘思路

黑盒挖掘: 测试一些敏感功能点, 特别关注后台高危功能点, 抓包看是否含有token参数, 如果没有, 再直接
请求这个页面, 不带 referer, 如果返回数据还是一样的, 那说明很有可能存在 CSRF 漏洞 .

白盒挖掘: 读取代码的核心文件, 查看里边有没有验证 token 和 referer 相关的代码 . 或者直接搜索 token 这个
关键字 , 再去看一下比较关心的功能点的代码有没有验证 . 

DESTOON 从 CSRF 到 GETSHELL: [https://bbs.ichunqiu.com/thread-27413-1-1.html](https://bbs.ichunqiu.com/thread-27413-1-1.html)

### GET 型 CSRF

修改 DVWA 安全等级为 Low, 然后到 CSRF 测试环境, 用户修改密码需要连续输入两次新密码, 

![image-20211222111848196](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222111848196.png)

在提交的的时候通过 BurpSuite 抓取数据包, 

![image-20211222113644328](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222113644328.png)

low 级别的 csrf 环境仅通过 GET 方法进行密码修改, 没有任何校验, 所以通过伪造符合要求的 URL, 诱使用户进行点击就可以做到任意密码的修改.

![image-20211222113007992](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222113007992.png)

使用 BurpSuite 的 PoC 生成工具可以快速生成测试页面 .

为了让受害者无法察觉到攻击或减少对攻击的怀疑, 可以对 PoC 进行改造 . 

* 删除 `<input type="submit" value="Submit request" />` 后页面无提交按钮 
* 添加`<script>document.forms[0].submit();` 自动提交请求, 访问页面直接发送请求

![image-20211222113939282](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222113939282.png)

诱导用户点击伪造的链接后密码被修改

![image-20211222113845636](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222113845636.png)

#### 突破 Referer 的检测

修改 DVWA 的安全等级为 Medium, 此时 CSRF 环境添加了对 Referer 字段的判断, 通过上面的方法使用 BurpSuite 生成的 CSRF HTML 无法修改密码.

![image-20211222151847590](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222151847590.png)

Referer 属性返回载入当前文档的来源文档的 URL, 如果当前文档不是通过超级链接访问的, 则为 null, 此属性允许客户端 JavaScript 访问 HTTP 引用头部.

![image-20211222150244580.png](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222150244580.png)

由于 Referer 字段可以通过客户端进行控制, 是不可信的字段, 所以在 BurpSuite 中通过 Repeater 发送带有 Referer 的请求或使用 HackBar 为浏览器请求添加 Referer 字段可以绕过,

尝试在 BurpSuite 生成的 CSRF HTML 中使用 JS 修改 Referer, 但未能成功 ...

#### 突破 Token 检测

修改 DVWA 的安全等级为 high, 发现请求中多了 user_token .

![image-20211222155848579](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222155848579.png)

此时发送修改密码的请求会提示 CSRF token 不正确, token 在计算机身份认证中表示临时令牌, 每次修改密码都会重置 token, 每次修改密码都会使用新的 token, 所以进行 CSRF 攻击必须获取到有效的 token 才可以 . 

![image-20211222160325054](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222160325054.png)

在 DVWA 中可以结合反射型 XSS 漏洞获取 CSRF 页面的 token .

```html
<iframe src="../csrf" onload=alert(frames[0].document.getElementsByName('user_token')[0].value)>
```

![image-20211222162150249](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222162150249.png)

使用 XSS 获得的 token, 配合 Burpsuite 生成的 CSRF HTML, 可以成功修改密码 . 

![image-20211222162626366](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222162626366.png)

### POST 型 CSRF 

使用 POST 方式的请求, 无法直接通过 URL 访问的方式触发 CSRF, 需要攻击者构造 POC, 利用其他方式触发漏洞 .

在 BurpSuite 中右键 Engagement tools --> Generate CSRF PoC 快速生成 CSRF HTML 页面 . 


## CSRF 漏洞利用



## CSRF 漏洞防护

