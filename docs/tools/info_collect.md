# 信息收集

## Google Hacking

* `site:`    `site:thief.one` 将返回所有和这个站有关的URL
* `inurl:`    搜索我们指定的字符是否存在于URL中
* `intitle:`    将返回所有网页标题中包含关键词的网页
* `intext:`    将返回所有在网页正文部分包含关键词的网页
* `cache:`    搜索Google里关于某些内容的缓存
* `define:`    搜索某个词语的定义
* `filetype:`    搜索指定的文件类型，如：bak,.mdb,.inc等
* `info:`    查找指定站点的一些基本信息
* `Link:`    `link:thief.one` 可以返回所有和thief.one做了链接的URL
* `index of:`    找目录遍历会用到
* `+`    强制包含某个字符进行查询
* `-`    查询的时候忽略某个字符
* `""`    查询的时候精确匹配双引号内的字符
* `.`    匹配某单个字符进行查询
* `*`    匹配任意字符进行查询
* `|`    或者, 表示多个选择, 只要有一个关键字匹配即可
* 注意: 在操作符, 冒号, 搜索项之间没有空格

一些例子

* `intitle:"Index of /ThinkPHP" | inurl:"/ThinkPHP/"`    寻找基于ThinkPHP 开发的应用
* `inurl:".php?id=" "you have an error in your sql syntax"`    寻找可能存在 SQL 注入漏洞的系统
* `inurl:/admin/upfile.asp`    寻找文件上传地址
* `inurl:.php? intext:CHARACTER_SETS,COLLATIONS,? intitle:phpmyadmin`    寻找未授权 phpmyadmin
* `filetype:xls 身份证 手机`    查找可能包含敏感信息的文件

Google Hacking Database

[https://www.exploit-db.com/google-hacking-database](https://www.exploit-db.com/google-hacking-database)

## 网络空间搜索引擎

网络空间搜索引擎是对全球网络空间基础设施或网络设备进行扫描, 并可以对指纹特征检索的平台, 通过扫描全网设备并抓取解析各个设备返回的信息. 可以对网络空间上的在线网络设备进行检索, 如服务器, 路由器, 交换机, 公共 IP 打印机, 网络摄像头等.

* [https://www.shodan.io/](https://www.shodan.io/)
* [https://fofa.so/](https://fofa.so/)
* [https://censys.io/](https://censys.io/)
* [https://www.zoomeye.org/](https://www.zoomeye.org/)

## 子域名信息收集

基于字典对子域名爆破枚举

* [subDomainsBrute](https://github.com/lijiejie/subDomainsBrute)
* 


第三方聚合 DNS 数据集

* [DNSdumpster](https://dnsdumpster.com/)
* [VirusTotal](https://www.virustotal.com/gui/home/search)
* [Netcraft](https://searchdns.netcraft.com/)

## 证书透明度 

授权机构(CA)是一个受信任的第三方组织, 负责发布和管理SSL/TLS证书, 全球有数百个受信任的CA, 他们中任何一个都有权利为你的域名颁发有效的SSL证书. 

证书透明度(CT)是为了防止证书授权机构(CA)或者其他恶意人员伪造服务器证书而诞生的一个项目. 

CT会要求CA将数字证书(SSL/TLS证书)公开并发布将颁发记录同步到日志服务器中. 而日志服务器则会提供给用户一个查找某域名颁发的所有数字证书途径。


数字证书中会包含子域名的相关信息. CT 日志只能增加, 不能减少, 所以证书上的子域名可能是过期的状态, 可以使用 masdns 工具对域名解析识别. 

CT 日志查询平台

* [crt.sh](https://crt.sh/)
* [Censys](https://search.censys.io/certificates)
* [Google透明度报告](https://transparencyreport.google.com/https/certificates)
* [Facebook透明度](https://developers.facebook.com/tools/ct)

## CSP

网页安全政策( Content Security Policy, 缩写CSP). CSP的实质就是白名单制度, 开发者明确告诉客户端, 哪些外部资源可以加载和执行, 等同于提供白名单. 

可以查看 HTTP 的 Content-Security-Policy 字段来搜集子域名信息. 