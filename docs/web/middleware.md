# 常见中间件及漏洞

| 中间件             | 漏洞              |
| ------------------ | ----------------- |
| IIS 常见漏洞 | IIS PUT 漏洞<br>IIS 短文件名猜解漏洞<br>IIS 解析漏洞<br>IIS 目录浏览漏洞 |
| Apche 常见漏洞 | Apache 解析漏洞<br>Apache 目录浏览漏洞 |
| Nginx 常见漏洞 | Nginx 解析漏洞<br>Nginx 目录浏览漏洞<br>Nginx 路径穿越漏洞 |
| Tomcat 常见漏洞 | Tomcat Basic 爆破<br>Tomcat 后门部署 war 文件<br>Tomcat 远程代码执行漏洞<br>Tomcat 文件读取漏洞 |
| Jboss 常见漏洞 | Jboss 反序列化漏洞<br>Jboss 后门部署 war 文件 |
| Weblogic 常见漏洞 | Weblogic 反序列化漏洞<br>Weblogic 后门部署 war 文件<br>Weblogic 任意文件上传<br>Weblogic SSRF 漏洞利用<br>Weblogic 文件密码解密的方式 |

| 默认信息 | Jboss | Tomcat | Weblogic |
| -------- | ----- | ------ | -------- |
| 默认端口 | 8080(Web),<br>9990(Console) | 8080(Web,Console) | 7001, 7002(Web, Console) |
| 默认口令 | jboss:jboss<br>admin:admin | tomcat:tomcat | weblogic:weblogic<br>system:system<br>portaladmin:portaladmin<br>guest:guest<br>weblogic:admin123<br>weblogic:weblogic123 |
| 默认路径 | `http://<host>:9990`(Console) | `http://<host>:8080/manager/html` | `http://<host>:7001/console/` |

