# Drupal XSS漏洞（CVE-2019-6341）

###### by ADummy

### 0x00利用路线

​			利用文件上传的漏洞上传图片中包含js代码--->用户访问url--->触发xss

### 0x01漏洞介绍

​			Drupal诞生于2000年，是一个基于PHP语言编写的开发型CMF（内容管理框架）。在某些情况下，通过文件模块或者子系统上传恶意文件触发XSS漏洞。

##### 			影响版本

```
Drupal <7.65
Drupal <8.6.13
Drupal <8.5.14
```



### 0x02漏洞复现

#### payload：[thezdi/PoC](https://github.com/thezdi/PoC/tree/master/Drupal)

​			**环境启动后，访问 `http://your-ip:8080/` 将会看到drupal的安装页面，一路默认配置下一步安装。因为没有mysql环境，所以安装的时候可以选择sqlite数据库。**

![Drupal_XSS漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_XSS漏洞_1.jpg)

​			**Drupal 的图片默认存储位置为 `/sites/default/files/pictures/<YYYY-MM>/`，默认存储名称为其原来的名称，所以之后在利用漏洞时，可以知道上传后的图片的具体位置。使用PoC上传构造好的伪造GIF文件**

```
php blog-poc.php 192.168.15.130
```

![Drupal_XSS漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_XSS漏洞_2.jpg)

​		**上传成功后，访问http://192.168.15.130:8080/sites/default/files/pictures/2021-02/_0图片位置，即可触发 XSS 漏洞，如下图所示**

![Drupal_XSS漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_XSS漏洞_3.jpg)



### 0x03参考资料

https://blog.csdn.net/ploto_cs/article/details/108535469





