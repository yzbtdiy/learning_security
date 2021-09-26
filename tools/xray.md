# Xray 漏洞扫描

## Xray 简介

Xray 是一款功能强大的安全评估工具, 有多名经验丰富的一线安全从业者打造而成. 

### 支持多种漏洞检测

```
XSS 跨站脚本攻击			SSRF 跨站请求伪造
命令, 代码注入			ThinkPHP 系列漏洞
路径穿越  				 弱口令
SQL 注入				  文件上传
目录枚举				 POC 框架
XML 实体注入			 CRLF 注入
```

POC 框架默认使用内置 Github 上贡献的 POC, 用户也可以根据需要自行构建 POC 并运行. 

漏洞扫描和运行时的状态统称为结果输出, Xray 定义了如下几种输出方式. 

* Stdout  标准输出    默认的输出方式
* JSON  文件输出    `--json-output FILENAME.json`
* HTML  报告输出    `--html-output FILENAME.json`
* Webhook 输出    `--webhook-output value`

```cmd
~ $ xray version

____  ___.________.    ____.   _____.___.
\   \/  /\_   __   \  /  _  \  \__  |   |
 \     /  |    _  _/ /  /_\  \  /   |   |
 /     \  |    |   \/    |    \ \____   |
\___/\  \ |____|   /\____|_   / / _____/
      \_/       \_/        \_/  \/

Version: 1.7.1/f725e41e/COMMUNITY

[xray 1.7.1/f725e41e]
Build: [2021-03-18] [linux/amd64] [RELEASE/COMMUNITY]
Compiler Version: go version go1.15.6 linux/amd64

To show open source licenses, please use `osslicense` sub-command.
```

## Xray 基本扫描模式

### 代理模式

作为客户端与服务器的中间人, 被动进行流量重发分析.

生成证书, 浏览器导入 ca.crt 至证书颁发机构

```cmd
~ $ xray genca
```

Xray 开启代理后, 浏览器设置代理将流量转发至 Xray 进行扫描

```cmd
~ $ xray webscan --listen 127.0.0.1:8888 --html-output proxy_scan.html
```

扫描结束 Ctrl+C 终止 Xray 任务

### 爬虫模式

模拟人工主动访问爬取链接, 自动进行扫描分析, 探测是否存在漏洞.

```cmd
~ $ xray webscan --basic-crawler http://testphp.vulnweb.com/ --html-output spider_scan.html
```

### 服务模式

使用预制 POC 扫描中间件等服务漏洞, 但目前内置 POC 较少(只有 tomcat-cve-2020-1938)

```cmd
~ $ xray servicescan --target testphp.vulnweb.com:80 --html-output service_scan.html
```

* `--target-file` 可以批量扫描, 格式为 `IP:PORT`, 一行一个

## 扫描插件

* `--plugins xss,sqldet,phantasm`  指定要运行的插件, 用 `,` 分隔
* `--poc`  配置本次扫描启用哪些 POC, 使用 `,` 分隔
* `--plugins phantasm --poc poc-yaml-thinkphp5-controller-rce`  只加载一个 POC, 精准匹配
* `--plugins phantasm --poc "*thinkphp*"`  加载内置所有带 thinkphp 的 POC
* `--plugins phantasm --poc "/home/test/pocs/*"`  加载本地 `/home/test/pocs` 目录中所有 POC
* `--plugins phantasm --poc "/home/test/pocs/*thinkphp*"`  加载 `/home/test/pocs` 下包含 thinkphp 的 POC

## 联动 BurpSuite

Xray 开启端口监听后 BurpSuite 中配置 Upstream Proxy Servers

BurpSuite --> User Options --> Upstream Proxy Server (配置目标地址以及 Xray 代理地址和端口)

浏览器代理配置为 BurpSuite 的 8080 端口, 实现 BurpSuite 和 Xray 的数据同步