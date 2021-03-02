# Weblogic 文件读取弱口令漏洞

###### by ADummy

### 0x00利用路线

​			后台弱口令、如果爆破不出来，可以用文件读取漏洞解密--->后台上传war包--->getshell

### 0x01漏洞介绍			

​			本环境模拟了一个真实的weblogic环境，其后台存在一个弱口令，并且前台存在任意文件读取漏洞。分别通过这两种漏洞，模拟对weblogic场景的渗透。

Weblogic版本：10.3.6(11g)

Java版本：1.6

### 0x02漏洞复现

本环境存在弱口令：

- weblogic
- Oracle@123

weblogic常用弱口令： http://cirt.net/passwords?criteria=weblogic

任意文件读取漏洞的利用

假设不存在弱口令，如何对weblogic进行渗透？

本环境前台模拟了一个任意文件下载漏洞，访问`http://your-ip:7001/hello/file.jsp?path=/etc/passwd`可见成功读取passwd文件。那么，该漏洞如何利用？

![Weblogic_文件读取弱口令漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_文件读取弱口令漏洞_1.jpg)

### 读取后台用户密文与密钥文件

weblogic密码使用AES（老版本3DES）加密，对称加密可解密，只需要找到用户的密文与加密时的密钥即可。这两个文件均位于base_domain下，名为`SerializedSystemIni.dat`和`config.xml`，在本环境中为`./security/SerializedSystemIni.dat`和`./config/config.xml`（基于当前目录`/root/Oracle/Middleware/user_projects/domains/base_domain`）。

`SerializedSystemIni.dat`是一个二进制文件，所以一定要用burpsuite来读取，用浏览器直接下载可能引入一些干扰字符。在burp里选中读取到的那一串乱码，右键copy to file就可以保存成一个文件：

![Weblogic_任意文件上传漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_任意文件上传漏洞_5.jpg)

解密成功，我没有java1.6环境，懒了。

![Weblogic_文件读取弱口令漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_文件读取弱口令漏洞_2.jpg)

![Weblogic_文件读取弱口令漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_文件读取弱口令漏洞_3.jpg)

获取到管理员密码后，登录后台。点击左侧的部署，可见一个应用列表：

点击安装，选择“上载文件”，上传war包。

成功获取webshell

![Weblogic_文件读取弱口令漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_文件读取弱口令漏洞_4.jpg)