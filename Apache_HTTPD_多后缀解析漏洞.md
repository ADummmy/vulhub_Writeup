## Apache HTTPD 多后缀解析漏洞

###### by ADummy

### 0x00利用路线

​			上传php文件--->Burpsuite抓包改包--->php文件上传成功--->getshell

### 0x01漏洞介绍

​			

##### 			影响版本Apache HTTPD 支持一个文件拥有多个后缀，并为不同后缀执行不同的指令。比如，如下配置文件：

```
AddType text/html .html
AddLanguage zh-CN .cn
```

​			其给`.html`后缀增加了media-type，值为`text/html`；给`.cn`后缀增加了语言，值为`zh-CN`。此时，如果用户请求文件`index.cn.html`，他将返回一个中文的html页面。

以上就是Apache多后缀的特性。如果运维人员给`.php`后缀增加了处理器：

```
AddHandler application/x-httpd-php .php
```

​			那么，在有多个后缀的情况下，只要一个文件含有`.php`后缀的文件即将被识别成PHP文件，没必要是最后一个后缀。利用这个特性，将会造成一个可以绕过上传白名单的解析漏洞。

##### 		影响版本

```
全版本
```

### 0x02漏洞复现

​			**上传一个名为1.php的文件，被拦截**

![Apache_HTTPD_多后缀解析漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_HTTPD_多后缀解析漏洞_1.jpg)

​			**在1.php后面插入一个.jpg，不再拦截**

![Apache_HTTPD_多后缀解析漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_HTTPD_多后缀解析漏洞_2.jpg)

​			**访问刚才上传的`/1.php.jpg`查看上传的文件,phpinfo被执行**

![Apache_HTTPD_多后缀解析漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_HTTPD_多后缀解析漏洞_3.jpg)

​			**上传一句话木马，蚁剑连接，getshell**

![Apache_HTTPD_多后缀解析漏洞_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_HTTPD_多后缀解析漏洞_4.jpg)

​			

### 0x03总结

​			没什么难度，直接搞就行

防御手段：

1、使用SetHandler,写好正则 <FileMatch ".+\.php$"> SetHandler application/x-httpd-php </FileMatch>

2、禁止.php这样的文件执行 <FileMatch ".+\.ph(p[3457]?|t|tml)\."> Require all denied </FileMatch>

3.将AddHandler application/x-httpd-php.php的配置文件删除



