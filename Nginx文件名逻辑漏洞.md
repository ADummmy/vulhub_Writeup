# Nginx 文件名逻辑漏洞（CVE-2013-4547）

###### by ADummy

### 0x00利用路线

​			上传文件后边空格--->上传文件成功--->php文件被成功解析

### 0x01漏洞介绍

​			这个漏洞其实和代码执行没有太大的关系,主要原因是错误地解析了请求的URL,错误地获取到用户请求的文件名,导致出现权限绕过、代码执行的连带影响。

##### 影响版本

```
Nginx  0.8.41 ~ 1.4.3 
Nginx  1.5.0 ~ 1.5.7
```

### 0x02漏洞复现

​			这个漏洞其实和代码执行没有太大关系，其主要原因是错误地解析了请求的URI，错误地获取到用户请求的文件名，导致出现权限绕过、代码执行的连带影响。

举个例子，比如，Nginx匹配到.php结尾的请求，就发送给fastcgi进行解析，常见的写法如下：

```
location ~ \.php$ {
    include        fastcgi_params;

    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  /var/www/html$fastcgi_script_name;
    fastcgi_param  DOCUMENT_ROOT /var/www/html;
}
```

正常情况下（关闭pathinfo的情况下），只有.php后缀的文件才会被发送给fastcgi解析。

而存在CVE-2013-4547的情况下，我们请求`1.gif[0x20][0x00].php`，这个URI可以匹配上正则`\.php$`，可以进入这个Location块；但进入后，Nginx却错误地认为请求的文件是`1.gif[0x20]`，就设置其为`SCRIPT_FILENAME`的值发送给fastcgi。

fastcgi根据`SCRIPT_FILENAME`的值进行解析，最后造成了解析漏洞。

所以，我们只需要上传一个空格结尾的文件，即可使PHP解析之。

再举个例子，比如很多网站限制了允许访问后台的IP：

```
location /admin/ {
    allow 127.0.0.1;
    deny all;
}
```

我们可以请求如下URI：`/test[0x20]/../admin/index.php`，这个URI不会匹配上location后面的`/admin/`，也就绕过了其中的IP验证；但最后请求的是`/test[0x20]/../admin/index.php`文件，也就是`/admin/index.php`，成功访问到后台。（这个前提是需要有一个目录叫`test`：这是Linux系统的特点，如果有一个不存在的目录，则即使跳转到上一层，也会爆文件不存在的错误，Windows下没有这个限制）

环境启动后，访问`http://your-ip:8080/`即可看到一个上传页面。

这个环境是黑名单验证，我们无法上传php后缀的文件，需要利用CVE-2013-4547。我们上传一个`1.gif`，注意后面的空格：

![Nginx文件名逻辑漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nginx文件名逻辑漏洞_1.jpg)

![Nginx文件名逻辑漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nginx文件名逻辑漏洞_2.jpg)

此处有一个坑点，就是访问文件的时候也需要改包，这就带来一个问题 蚁剑连不上shell，希望有师傅能解决这个问题。