# 带有PHPINFO的PHP本地文件包含RCE

###### by ADummy

### 0x00利用路线

​			执行exp--->getshell

​			适用于有本地文件包含漏洞，有phpinfo，怎么写入webshell的情况

### 0x01漏洞介绍			

​			在PHP文件包含漏洞中，当我们找不到要触发RCE的有效文件时，如果有PHPINFO可以告诉我们随机生成的临时文件名及其位置，则可能可以包含一个临时文件来利用它来利用它。

启动环境后，可访问`http://your-ip:8080/phpinfo.php`以获取PHPINFO页面并`http://your-ip:8080/lfi.php?file=/etc/passwd`显示存在LFI漏洞。

## 利用细节

​			当向php发送POST请求并且该请求包含FILE块时，PHP会将发布的文件保存到一个临时文件中（通常为`/tmp/php[6 random digits]`），该文件名可以在`$_FILES`变量中找到。请求结束后，该临时文件将被删除。

​			同时，PHPINFO页面显示上下文中的所有变量，包括`$_FILES`。因此，如果我们将POST请求发送到PHPINFO页面，则可以在响应中找到临时文件的名称。

这样，就可以将LFI漏洞升级为RCE，而无需存在可用的本地文件。

​			文件包含和PHPINFO通常位于不同的网页中。从理论上讲，我们需要在文件上传请求到PHPINFO页面的响应中检索文件名之后，才将文件名发送到文件包含页面。但是，在第一个请求完成后，该文件将从磁盘中删除，因此我们需要赢得比赛。

脚步：

1. 将文件上传请求发送到PHPINFO页面，其中HEADER和GET字段填充了大块垃圾数据。
2. 响应内容将是巨大的，因为PHPINFO将打印出所有数据。
3. PHP的默认输出缓冲区大小为4096字节。可以理解为PHP在套接字连接期间每次返回4096字节。
4. 因此，我们使用原始套接字来实现我们的目标。每次我们读取4096个字节并将文件名发送到LFI页面时，就将其发送给LFI页面。
5. 到我们获得文件名时，第一个套接字连接尚未结束，这意味着临时文件在那时仍然存在。
6. 通过利用时间间隔，可以包含并执行临时文件

### 0x02漏洞复现

python脚本[exp.py](https://github.com/vulhub/vulhub/blob/master/php/inclusion/exp.py)实现了上述过程。成功包含临时文件后，`<?php file_put_contents('/tmp/g', '<?=eval($_REQUEST[1])?>')?>`将执行以生成永久文件`/tmp/g`以供进一步使用。

![PHP_LFI_RCE_利用phpinfo_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_LFI_RCE_利用phpinfo_1.jpg)

使用python2 `python exp.py your-ip 8080 100`：

![PHP_LFI_RCE_利用phpinfo_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_LFI_RCE_利用phpinfo_2.png)

文件写入成功，蚁剑连接

![PHP_LFI_RCE_利用phpinfo_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_LFI_RCE_利用phpinfo_3.jpg)