# Jupyter Notebook 未授权访问漏洞

###### by ADummy

### 0x00利用路线

​			后台无密码，直接访问。

### 0x01漏洞介绍

​			Jupyter Notebook（此前被称为 IPython notebook）是一个交互式笔记本，支持运行 40 多种编程语言。

如果管理员未为Jupyter Notebook配置密码，将导致未授权访问漏洞，游客可在其中创建一个console并执行任意Python代码和命令

### 0x02漏洞复现

运行后，访问`http://your-ip:8888`将看到Jupyter Notebook的Web管理界面，并没有要求填写密码。

选择 new -> terminal 即可创建一个控制台：

![Jupyter_Notebook_Unauth_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Jupyter_Notebook_Unauth_RCE_1.jpg)

![Jupyter_Notebook_Unauth_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Jupyter_Notebook_Unauth_RCE_2.jpg)













