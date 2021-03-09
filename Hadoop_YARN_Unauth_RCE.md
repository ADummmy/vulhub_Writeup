# Hadoop YARN ResourceManager 未授权访问

###### by ADummy

### 0x00利用路线

​			exp直接打--->其实是发了两次http包

### 0x01漏洞介绍

​		访问`http://your-ip:8088`即可看到Hadoop YARN ResourceManager WebUI页面

![Hadoop_YARN_Unauth_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Hadoop_YARN_Unauth_RCE_1.jpg)

​		本机监听9999端口，收到shell	

![Hadoop_YARN_Unauth_RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Hadoop_YARN_Unauth_RCE_2.jpg)

### 0x03参考资料

https://blog.csdn.net/xuandao_ahfengren/article/details/107127276

