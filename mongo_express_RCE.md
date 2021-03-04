

# mongo-express 远程代码执行漏洞（CVE-2019-10758）

###### by ADummy

### 0x00利用路线

​			Burpsuite发包--->命令执行

### 0x01漏洞介绍

​		mongo-express是一款mongodb的第三方Web界面，使用node和express开发。如果攻击者可以成功登录，或者目标服务器没有修改默认的账号密码（`admin:pass`），则可以执行任意node.js代码。

### 0x 02漏洞复现

![mongo_express_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/mongo_express_RCE_1.jpg)



![mongo_express_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/mongo_express_RCE_2.jpg)

![mongo_express_RCE_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/mongo_express_RCE_3.jpg)





