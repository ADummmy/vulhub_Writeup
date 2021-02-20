## Apache HTTPD 换行解析漏洞

###### by ADummy

### 0x00利用路线

​			上传php文件--->Burpsuite抓包改包--->php文件上传成功--->getshell

### 0x01漏洞介绍

​			Apache HTTPD是一款HTTP服务器，它可以通过mod_php来运行PHP网页。其2.4.0~2.4.29版本中存在一个解析漏洞，在解析PHP时，`1.php\x0A`将被按照PHP后缀进行解析，导致绕过一些服务器的安全策略

##### 			影响版本

```
apache httpd 2.4.0~2.4.29
```

### 0x02漏洞复现

​			**上传一个名为1.php的文件，被拦截**

![Apache_HTTPD_换行解析漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_HTTPD_换行解析漏洞_1.jpg)

​			**在1.php后面插入一个`\x0A`（注意，不能是`\x0D\x0A`，只能是一个`\x0A`），不再拦截**

![Apache_HTTPD_换行解析漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_HTTPD_换行解析漏洞_2.jpg)

​			**访问刚才上传的`/1.php%0a`查看上传的文件,phpinfo被执行**

![Apache_HTTPD_换行解析漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_HTTPD_换行解析漏洞_3.jpg)

​			**上传一句话木马，蚁剑连接，getshell**

![Apache_HTTPD_换行解析漏洞_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_HTTPD_换行解析漏洞_4.jpg)

​			

### 0x03总结

​			没什么难度，直接搞就行



