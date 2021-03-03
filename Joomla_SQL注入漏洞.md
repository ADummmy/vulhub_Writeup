

# Joomla 3.7.0 (CVE-2017-8917) SQL注入漏洞环境

###### by ADummy

### 0x00利用路线

​			url注入

### 0x01漏洞介绍

​			漏洞是由此Joomla 3.7.0 版本中引入的新组件（com_fields）带来的

### 0x02漏洞复现

	安装joomla，
	数据库地址：mysql:3306
	用户：root
	密码：root
	数据库名：joomla
	填入上述信息，正常安装即可。
![Joomla_SQL注入漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Joomla_SQL注入漏洞_1.jpg)

![Joomla_SQL注入漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Joomla_SQL注入漏洞_2.jpg)

