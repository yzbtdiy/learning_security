# Burpsuite 简介

BurpSuite是一个集成化的渗透测试工具，它集合了多种渗透测试组件，使我们自动化地或手工地能更好的完成对web应用的渗透测试和攻击。在渗透测试中，我们使用Burp Suite将使得测试工作变得更加容易和方便，即使在不需要娴熟的技巧的情况下，只有我们熟悉Burp Suite的使用，也使得渗透测试工作变得轻松和高效。

BurpSuite基于Java开发, 新版本要求Java9及以上JDK, 建议使用JDK11

## JDK 下载

除了Oracle官方的JDK以外还有许多基于OpenJDK衍生出来的其他版本, 如corretto-jdk(亚马逊), dragonwell-jdk(阿里), microsoft-jdk(微软)等众多厂商提供的衍生版JDK

推荐微软的JDK11: [https://aka.ms/download-jdk/microsoft-jdk-11.0.13.8.1-windows-x64.msi](https://aka.ms/download-jdk/microsoft-jdk-11.0.13.8.1-windows-x64.msi)


## Burpsuite 下载

新版BurpSuite官网已开放下载: [https://portswigger.net/Burp/Releases](https://portswigger.net/Burp/Releases)


## 注册机

注册机可在Github上查找, 推荐: [https://github.com/h3110w0r1d-y/BurpLoaderKeygen](https://github.com/h3110w0r1d-y/BurpLoaderKeygen)

## bat批处理

```
@echo off
start "burpsuite" /B javaw.exe -javaagent:BurpLoaderKeygen.jar -noverify -jar burpsuite_pro.jar
exit 0 
```

## 吾爱破解爱盘下载

吾爱破解论坛的爱盘提供了内置授权文件的Brupsuite的老版本和最新版的压缩包, 可以直接下载使用: [https://down.52pojie.cn/Tools/Network_Analyzer/](https://down.52pojie.cn/Tools/Network_Analyzer/)

