# Gitlab 任意文件读取漏洞（CVE-2016-9086）

###### by ADummy

### 0x00利用路线

​			上传poc、tar.gz文件--->读取到etc/passwd

### 0x01漏洞介绍

​			2018年底来自Semmle Security Research Team的Man Yue Mo发表了CVE-2018-16509漏洞的变体CVE-2018-19475，可以通过一个恶意图片绕过GhostScript的沙盒，进而在9.26以前版本的gs中执行任意命令。

### 0x02漏洞复现

环境运行后，Web端口为10080，ssh端口为10022。访问`http://your-ip:10080`，设置管理员（用户名`root`）密码，登录。

新建一个项目，点击`GitLab export`

![Gitlab_任意文件读取漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Gitlab_任意文件读取漏洞_1.jpg)

将[test.tar.gz](https://github.com/vulhub/vulhub/blob/master/gitlab/CVE-2016-9086/test.tar.gz)上传，将会读取到`/etc/passwd`文件内容：

![Gitlab_任意文件读取漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Gitlab_任意文件读取漏洞_2.jpg)

