# influxdb未授权访问漏洞

###### by ADummy

### 0x00利用路线

​			Influxdb http query接口---> 生成jwt凭证绕过认证 --->任意sql语句执行

### 0x01漏洞介绍

​			InfluxDB是一个使用Go语言编写的开源分布式，支持高并发的时序数据库，其使用JWT作为鉴权方式。在用户开启了认证，但未设置参数shared-secret的情况下，JWT的认证密钥为空字符串，此时攻击者可以伪造任意用户身份在InfluxDB中执行SQL语句。

​			影响版本

  		Discuz < 1.7.6

### 0x02漏洞复现

**访问`http://your-ip:8086/debug/vars`即可查看一些服务信息，但此时执行SQL语句则会出现401错误**

![influxdb未授权访问漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/influxdb未授权访问漏洞_2.jpg)



**我们借助https://jwt.io/来生成jwt token**

**{  "alg": "HS256",  "typ": "JWT" } {  "username": "admin",  "exp": 1676346267 }**

![influxdb未授权访问漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/influxdb未授权访问漏洞_1.jpg)





**发送带有这个jwt token的数据包，可见SQL语句执行成功**

![influxdb未授权访问漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/influxdb未授权访问漏洞_3.jpg)

​				
### 0x03参考资料

https://blog.csdn.net/u010918487/article/details/99541683

https://www.colabug.com/2020/0102/6801890/
