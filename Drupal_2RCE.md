# Drupal 远程代码执行漏洞（CVE-2019-6339）

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->信息被爆出

### 0x01漏洞介绍

​			Drupal 是一款用量庞大的CMS，其7.0~7.31版本中存在一处无需认证的SQL漏洞。通过该漏洞，攻击者可以执行任意SQL语句，插入、修改管理员信息，甚至执行任意代码。

### 0x02漏洞复现

#### payload：[thezdi/PoC](https://github.com/thezdi/PoC/tree/master/Drupal)

​			**环境启动后，访问 `http://your-ip:8080/` 将会看到drupal的安装页面，一路默认配置下一步安装。因为没有mysql环境，所以安装的时候可以选择sqlite数据库。**

![Drupal_2RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_2RCE_1.jpg)

​			**默认页面**

![Drupal_2RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_2RCE_2.jpg)



​			**Drupal 的图片默认存储位置为 /sites/default/files/pictures/<YYYY-MM>/，默认存储名称为其原来的名称，所以之后在利用漏洞时，可以知道上传后的图片的具体位置。**
**访问 http://127.0.0.1:8080/admin/config/media/file-system，在 Temporary directory 处输入之前上传的图片路径，示例为 phar://./sites/default/files/pictures/2021-02/blog-ZDI-CAN-7232-cat_0.jpg，保存后将触发该漏洞。如下图所示，触发成功。**

![Drupal_2RCE_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_2RCE_3.jpg)

![Drupal_2RCE_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_2RCE_4.jpg)		

​		**对poc进行分析**

![Drupal_2RCE_5](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_2RCE_5.jpg)



### 0x03参考资料

https://www.cnblogs.com/mke2fs/p/12631677.html





