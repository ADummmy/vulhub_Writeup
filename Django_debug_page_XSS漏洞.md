# Django debug page XSS漏洞(CVE-2017-12794)

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->用户创建成功--->成功登录

### 0x01漏洞介绍

​			Django默认配置下，如果匹配上的URL路由中最后一位是/，而用户访问的时候没加/，Django默认会跳转到带/的请求中。（由配置项中的`django.middleware.common.CommonMiddleware`、`APPEND_SLASH`来决定）。在path开头为`//example.com`的情况下，Django没做处理，导致浏览器认为目的地址是绝对路径，最终造成任意URL跳转漏洞。

​			如果`django.middleware.common.CommonMiddleware`和`APPEND_SLASH`设置都启用，并且项目的URL模式可以接受以斜杠结尾的任何路径，则对该站点恶意制作的URL的请求可能导致重定向到另一个站点，从而启用网络钓鱼和其他攻击。

##### 影响版本

```
Django < 1.11.5
```

### 0x02漏洞复现



1. **用户注册页面，未检查用户名**
2. **注册一个用户名为`<script>alert(1)</script>`的用户**
3. **再次注册一个用户名为`<script>alert(1)</script>`的用户**
4. **触发duplicate key异常，导致XSS漏洞**



访问`http://your-ip:8000/create_user/?username=<script>alert(1)</script>`创建一个用户，成功

![Django_debug_page_XSS漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Django_debug_page_XSS漏洞_1.jpg)

再次访问`http://your-ip:8000/create_user/?username=<script>alert(1)</script>`，触发异常

![Django_debug_page_XSS漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Django_debug_page_XSS漏洞_2.jpg)



可见，Postgres抛出的异常为

```
duplicate key value violates unique constraint "xss_user_username_key"
DETAIL:  Key (username)=(<script>alert(1)</script>) already exists.
```

这个异常被拼接进`The above exception ({{ frame.exc_cause }}) was the direct cause of the following exception`，最后触发XSS。