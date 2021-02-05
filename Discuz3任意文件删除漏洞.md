# Discuz!X ≤3.4 任意文件删除漏洞

###### by ADummy

### 0x00利用路线

​			Discuz个人设置界面--->burpsuite抓包改包--->上传文件替换脏数据--->导致文件删除

### 0x01漏洞介绍

​			通过配置个人信息的属性值，导致文件删除。

​			影响版本

  		Discuz <= 3.4

### 0x02漏洞复现

​			安装Discuz，数据库地址为**db**

![Discuz_filedelete_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_filedelete_1.jpg)

​			查看robots文件，此时文件存在

![Discuz_filedelete_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_filedelete_2.jpg)

​			登录，并进去设置页面，点击保存并抓包，formhash值记一下。

![Discuz_filedelete_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_filedelete_3.jpg)

​			可以看到出生地被修改(笔者做了多次试验，上图和下图驴唇不对马嘴。时间有限，抱歉)

![Discuz_filedelete_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_filedelete_4.jpg)

​			写一个html网页，目的是为了上传一个文件，使文件被删除。

![Discuz_filedelete_6](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_filedelete_6.jpg)

​	再次查看robots文件，发现文件已经被删除。

![Discuz_filedelete_7](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_filedelete_7.png)



### 0x03总结

​				整体复现较为简单。坑也比较少，容易出现失误的情况就是 hash值有可能被刷新，不关闭浏览器页面，成功率会高一些。

### 0x04参考资料

https://paper.seebug.org/411/