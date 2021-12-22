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

### GET 型 CSRF

修改 DVWA 安全等级为 Low, 然后到 CSRF 测试环境, 用户修改密码需要连续输入两次新密码, 

![image-20211222111848196](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222111848196.png)

在提交的的时候通过 BurpSuite 抓取数据包, 

![image-20211222113644328](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222113644328.png)

low 级别的 csrf 环境仅通过 GET 方法进行密码修改, 没有任何校验, 所以通过伪造符合要求的 URL, 诱使用户进行点击就可以做到任意密码的修改.

![image-20211222113007992](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222113007992.png)

![image-20211222113939282](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222113939282.png)

诱导用户点击伪造的链接后密码被修改

![image-20211222113845636](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/csrf/image-20211222113845636.png)

### 突破 Referer 的检测

修改 DVWA 的安全等级为 Medium, 此时 CSRF 环境添加了对 Referer 字段的判断.

Referer 属性返回载入当前文档的来源文档的 URL, 如果当前文档不是通过超级链接访问的, 则为 null, 此属性允许客户端 JavaScript 访问 HTTP 引用头部.




## CSRF 漏洞利用



## CSRF 漏洞防护

