# PHPMailer 任意文件读取漏洞（CVE-2017-5223）

###### by ADummy

### 0x00利用路线

​			注册邮箱--->配置环境--->插入恶意代码--->目标服务器发送邮件--->服务器文件被读出

### 0x01漏洞介绍			

​			PHPMailer在发送邮件的过程中，会在邮件内容中寻找图片标签（`<img src="...">`），并将其src属性的值提取出来作为附件。所以，如果我们能控制部分邮件内容，可以利用`<img src="/etc/passwd">`将文件`/etc/passwd`作为附件读取出来，造成任意文件读取漏洞。

在当前目录下创建文件`.env`，内容如下（将其中的配置值修改成你的smtp服务器、账户、密码）：

```
SMTP_SERVER=smtp.example.com
SMTP_PORT=587
SMTP_EMAIL=your_email@example.com
SMTP_PASSWORD=secret
SMTP_SECURE=tls
```

其中，`SMTP_SECURE`是SMTP加密方式，可以填写none、ssl或tls。

##### 影响版本

```
PHPMailer <= 5.2.21
```

### 0x02漏洞复现

意见反馈”页面，正常用户填写昵称、邮箱、意见提交，这些信息将被后端储存，同时后端会发送一封邮件提示用户意见填写完成：

```
该场景在实战中很常见，比如用户注册网站成功后，通常会收到一封包含自己昵称的通知邮件，那么，我们在昵称中插入恶意代码`<img src="/etc/passwd">`，目标服务器上的文件将以附件的形式被读取出来。
```

我们填写恶意代码在“意见”的位置：

payload

```
<img src="/etc/passwd">
```

![PHPMailer_任意文件读取漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHPMailer_任意文件读取漏洞_1.jpg)

![PHPMailer_任意文件读取漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHPMailer_任意文件读取漏洞_2.jpg)



![PHPMailer_任意文件读取漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHPMailer_任意文件读取漏洞_3.jpg)

