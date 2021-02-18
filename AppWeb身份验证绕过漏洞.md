

# AppWeb身份验证绕过漏洞

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->改包--->获取session--->改包，使用session--->身份验证绕过

### 0x01漏洞介绍

​		AppWeb是Embedthis Software LLC公司负责开发维护的一个基于GPL开源协议的嵌入式Web Server。他使用C/C++来编写，能够运行在几乎先进所有流行的操作系统上。当然他最主要的应用场景还是为嵌入式设备提供Web Application容器。

AppWeb可以进行认证配置，其认证方式包括以下三种：

- 1.basic 传统HTTP基础认证

- 2.digest 改进版HTTP基础认证，认证成功后将使用Cookie来保存状态，而不用再传递Authorization头

- 3.form 表单认证

  **由于逻辑缺陷，使用伪造的HTTP请求，可以绕过HTTP form和HTTP digest登录类型的身份验证。**

​			影响版本

  		Appweb <= 7.0.2

### 0x02漏洞复现

​			**登录界面**

![AppWeb身份验证绕过漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/AppWeb身份验证绕过漏洞_1.jpg)

​		**抓包 修改Authorization字段的值 需要知道用户名，所以此漏洞利用的比较局限**

​		**可以看到 返回了session的值。**

![AppWeb身份验证绕过漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/AppWeb身份验证绕过漏洞_2.png)



**带着返回的session值访问，发现身份验证已经绕过，我们从来没输入过admin的密码。**

![AppWeb身份验证绕过漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/AppWeb身份验证绕过漏洞_3.jpg)

### 0x03参考资料

https://blog.csdn.net/weixin_42936566/article/details/87120710

https://blog.51cto.com/14259144/2420821