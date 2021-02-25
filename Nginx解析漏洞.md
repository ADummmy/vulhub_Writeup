# Nginx 解析漏洞

###### by ADummy

### 0x00利用路线

​			上传图片木马--->url后缀➕/.php--->访问url--->图片被当成php解析

### 0x01漏洞介绍			

​			漏洞与Nginx、php版本无关，属于用户配置不当造成的解析漏洞。

### 0x02漏洞复现

访问`http://your-ip/uploadfiles/nginx.png`和`http://your-ip/uploadfiles/nginx.png/.php`即可查看效果。

正常显示

![Nginx解析漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nginx解析漏洞_1.jpg)

访问/.php

![Nginx解析漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nginx解析漏洞_2.jpg)

自己做了个图片木马上传：

![Nginx解析漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nginx解析漏洞_3.jpg)