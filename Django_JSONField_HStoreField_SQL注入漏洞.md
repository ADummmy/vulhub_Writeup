# Django JSONField / HStoreField SQL注入漏洞（CVE-2019-14234）

###### by ADummy

### 0x00利用路线

​			直接url注入

### 0x01漏洞介绍

```
**Django是一个大而全的Web框架，其支持很多数据库引擎，包括Postgresql、Mysql、Oracle、Sqlite3等，但与Django天生为一对儿的数据库莫过于Postgresql了，Django官方也建议配合Postgresql一起使用。**
```

​		该漏洞需要开发者使用了JSONField/HStoreField，且用户可控queryset查询时的键名，在键名的位置注入SQL语句。Django通常搭配postgresql数据库，而JSONField是该数据库的一种数据类型。该漏洞的出现的原因在于Django中JSONField类的实现，Django的model最本质的作用是生成SQL语句，而在Django通过JSONField生成sql语句时，是通过简单的字符串拼接。通过JSONField类获得KeyTransform类并生成sql语句的位置。

##### 影响版本

```
Django < 1.11.23
Django < 2.1.11
Django < 2.2.4
```

### 0x02漏洞复现

##### 			环境启动后，访问`http://your-ip:8000`即可看到Django默认首页。

![Django_JSONField_HStoreField_SQL注入漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Django_JSONField_HStoreField_SQL注入漏洞_1.jpg)

**首先登陆后台`http://your-ip:8000/admin/`，用户名密码为`admin`、`a123123123`。**

**登陆后台后，进入模型`Collection`的管理页面`http://your-ip:8000/admin/vuln/collection/`**

![Django_JSONField_HStoreField_SQL注入漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Django_JSONField_HStoreField_SQL注入漏洞_2.jpg)

**然后在GET参数中构造`detail__a'b=123`提交，其中`detail`是模型`Collection`中的JSONField：**

**http://your-ip:8000/admin/vuln/collection/?detail__a%27b=123**

**可见，单引号已注入成功，SQL语句报错**

![Django_JSONField_HStoreField_SQL注入漏洞_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Django_JSONField_HStoreField_SQL注入漏洞_3.jpg)

### 0x03参考资料

https://www.cnblogs.com/huangxiaosan/p/14322147.html (强烈推荐，写的十分好且有拓展)



