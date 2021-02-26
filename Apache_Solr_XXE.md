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

不会xml，以后补充

### 0x03参考资料

https://blog.csdn.net/csacs/article/details/88220227

https://paper.seebug.org/425/

https://www.jianshu.com/p/35f9b91de524/