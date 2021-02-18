# DNS域传送漏洞

###### by ADummy

### 0x00利用路线

​			**nslookup手工验证**

​			**dig工具使用**

​			**dnsenum工具使用**

​			**nmap脚本扫描**

### 0x01漏洞介绍

​		**DNS** ：Domain Name System
一个保存IP地址和域名相互**映射**关系的**分布式**数据库，重要的互联网基础设施，默认使用的TCP/UDP端口号是**53**

常见DNS记录类型:

```
A       IP地址记录,记录一个域名对应的IP地址
AAAA    IPv6 地址记录，记录一个域名对应的IPv6地址
CNAME   别名记录，记录一个主机的别名 
MX      电子邮件交换记录，记录一个邮件域名对应的IP地址，如root@xxxx.com
NS      域名服务器记录 ,记录该域名由哪台域名服务器解析
PTR     反向记录，也即从IP地址到域名的一条记录
TXT     记录域名的相关文本信息1234567
```

------

**域传送** ：DNS Zone Transfer

DNS服务器分为：**主服务器**、**备份服务器**和**缓存服务器**。
**域传送**是指后备服务器从主服务器拷贝数据，并用得到的数据更新自身数据库。
在主备服务器之间同步数据库，需要使用“DNS域传送”。

**若DNS服务器配置不当，可能导致匿名用户获取某个域的所有记录。造成整个网络的拓扑结构泄露给潜在的攻击者，包括一些安全性较低的内部主机，如测试服务器。凭借这份网络蓝图，攻击者可以节省很少的扫描时间。**

**大的互联网厂商通常将内部网络与外部互联网隔离开，一个重要的手段是使用Private DNS。如果内部DNS泄露，将造成极大的安全风险。风险控制不当甚至造成整个内部网络沦陷。**

### 0x02漏洞复现

​		**nslookup手工验证**

​		nslookup进入交互式界面，server设置dns服务器，ls vulhub.org 显示dns服务器上vulhub.org所有子域名

![DNS域传送漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/DNS域传送漏洞_1.png)

​		**dig工具使用**

​		dig   @dns服务器 axfr -t vulhub.org

![DNS域传送漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/DNS域传送漏洞_2.jpg)

​		**dnsenum工具使用**(很强大的一款工具，使用自行百度)

![DNS域传送漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/DNS域传送漏洞_3.jpg)

​		**nmap脚本扫描**

​		nmap --script dns-zone-transfer.nse --script-args “dns-zone-tranfer.domain=vulhub.org” -Pn -p 53  dns服务器

![DNS域传送漏洞_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/DNS域传送漏洞_4.jpg)

### 0x03参考资料

http://www.lijiejie.com/dns-zone-transfer-1/

https://blog.csdn.net/c465869935/article/details/53444117