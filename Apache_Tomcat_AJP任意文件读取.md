

# Apache Tomcat AJP任意文件读取/包含漏洞（CVE-2020-1938）

###### by ADummy

### 0x00利用路线

​			exp直接利用

### 0x01漏洞介绍

​			Java是当前Web开发中最流行的编程语言，而Tomcat是最流行的Java中间件服务器之一。自最初发布以来，它已经使用了20多年。

[Ghostcat](https://www.chaitin.cn/en/ghostcat)是Chaitin Tech的安全研究人员发现的Tomcat中的一个严重漏洞。由于Tomcat AJP协议中的缺陷，攻击者可以读取或包括Tomcat的webapp目录中的任何文件。例如，攻击者可以阅读WebApp配置文件或源代码。此外，如果目标Web应用程序具有文件上传功能，则攻击者可能通过利用Ghostcat漏洞利用文件包含功能在目标主机上执行恶意代码。

### 0x 02漏洞复现

成功运行上述命令后，您将通过访问站点看到Tomcat的示例页面`http://your-ip:8080`，还有一个正在侦听的AJP端口8009。

![Apache_Tomcat_AJP任意文件读取_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_Tomcat_AJP任意文件读取_2.jpg)

![Apache_Tomcat_AJP任意文件读取_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_Tomcat_AJP任意文件读取_1.jpg)



