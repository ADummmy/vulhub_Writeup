

# JBoss JMXInvokerServlet 反序列化漏洞

###### by ADummy

### 0x00利用路线

​			ysoserial生成payload--->上传给网站进行解析--->命令执行

### 0x01漏洞介绍

​			Spring Data REST是一个构建在Spring Data之上，为了帮助开发者更加容易地开发REST风格的Web服务。在REST API的Patch方法中（实现[RFC6902](https://tools.ietf.org/html/rfc6902)），path的值被传入`setValue`，导致执行了SpEL表达式，触发远程命令执行漏洞。

​			影响版本

  	JBoss Enterprise Application Platform 6.4.4,5.2.0,4.3.0_CP10
  	JBoss AS (Wildly) 6 and earlier
  	JBoss A-MQ 6.2.0
  	JBoss Fuse 6.2.0
  	JBoss SOA Platform (SOA-P) 5.3.1
  	JBoss Data Grid (JDG) 6.5.0
  	JBoss BRMS (BRMS) 6.1.0
  	JBoss BPMS (BPMS) 6.1.0
  	JBoss Data Virtualization (JDV) 6.1.0
  	JBoss Fuse Service Works (FSW) 6.0.0
  	JBoss Enterprise Web Server (EWS) 2.1,3.0

访问http://your-ip:8080/invoker/JMXInvokerServlet只要有文件下载，目标网站即存在JBoss JMXInvokerServlet 反序列化漏洞

### 0x02漏洞复现

​	JBoss在处理`/invoker/JMXInvokerServlet`请求的时候读取了对象，所以我们直接将[ysoserial](https://github.com/frohoff/ysoserial)生成好的POC附在POST Body中发送即可。整个过程可参考[jboss/CVE-2017-12149](https://github.com/vulhub/vulhub/tree/master/jboss/CVE-2017-12149)，我就不再赘述。网上已经有很多EXP了，比如[DeserializeExploit.jar](https://cdn.vulhub.org/deserialization/DeserializeExploit.jar)，直接用该工具执行命令、上传文件即可：

```
java -jar ysoserial.jar CommonsCollections5 "touch /tmp/success" > poc.ser
```

几种方法都试过 感觉不太好用、 可能是linux和windows系统的问题。

![JBoss_JMXInvokerServlet_反序列化漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/JBoss_JMXInvokerServlet_反序列化漏洞_1.jpg)

