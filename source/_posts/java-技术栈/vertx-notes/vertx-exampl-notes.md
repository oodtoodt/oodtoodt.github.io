---
title: vert.x-exampl-notes
date: 2020-04-17 11:06:55
tags:
---


在把文档全部复制粘贴了之后，我意识到这根本没啥用。或者说用途有限，留下一个初步印象。
来读example代码，写点东西记一记

ongoing
<!--more-->

### idea
会经常性的抽风

## Net/Http

### net 和 http
NetServer、NetClient
HttpServer、HttpClient
服务器方面
net用到了Pump.pump(sock,sock).start()，http用的是req.response().putheader().end();
net是connectHandler ，http是requestHandler

客户端
net是connect，socket = res.result();,然后用socket操作,socket.handler
http直接是getNow,resp就可以操作了，.

当然，echo的功能是回声，而http-simple的功能就只是显示hello world
所以pump(rs,ws)，基本就是读取然后回复了

### ssl（https）
两种ssl在根本上差不多，XXClientOptions().setSsl().setTrustAll()，当然这种不安全，应当配置turst store和包含该客户端信任的服务器证书

### proxy
这就真的...套了好多层了，
```java
vertx.createHttpServer().requestHander(req->{
    ...
    HttpClientRequest c_req = client.request(req.method(),8282,"localhost",req.uri(),c_res->{
        ...
        c_res.handler(data->{
            ...
            req.response().write(data);
        });
        c_res.endHandler((v) -> req.response().end());
    });
    c_req.setChunked();
    c_req.headers().setAll(req.headers());
    req.handler(data->{
        ...
        c_req.write(data);
    });
    req.endHandler((v) -> c_req.end());
}).lesten(8080);
```
还分了两种，一种是用了ProxyOptions,用套接字连接端点服务器然后传递其他事件的，一种看起来简单一些，就是个代理接受请求。

### WebSocket
老实说这个例子挺失败的...说是双工但是怎么看都接受不到东西，我也不知道是html写的有问题还是这里的vertx写的有问题，总之，我没成功。那也没啥好探究了的（

### 本地文件
+ 如何用本地html
这个简单

### 表单 
+ 如何处理HTML form
虽然很简单，但是顺序很奇怪，像是被sort过，也有可能是异步的原因（但是每次都一样的，2143

### SimpleFormUploadServer.java
+ 如何处理form upload
这不就是我们那个10.xx里面有个奇怪文件夹嘛，一模一样的

### upload
这个是告诉你怎么直接泵送的，上下各写了一个Pump。
但我还是不怎么明白：这东西有什么用？对大文件直接落到本地的.uploaded文件，然后呢？

### sharing
+ 如何sharing 让多台服务器在同一主机和端口上侦听。尝试部署一个以上的实例。
是关于怎么跑很多核的，无非就是set了一下Instances，然后这里是用轮询的方式分发给任意一个处理的（传入新连接的时候）

### HTTP2 s1mple
和ssl差不太多，但如果你记得h2和h2c的话，你就会知道这个.setUseAlpn的含义了。
>ALPN是一个TLS的扩展，它在客户端和服务器开始交换数据之前协商协议。不支持ALPN的客户端仍然可以执行经典的SSL握手。通常情况，ALPN会对 h2 协议达成一致，尽管服务器或客户端决定了仍然使用 HTTP/1.1 协议。
要处理 h2c 请求，TLS必须被禁用，服务器将升级到 HTTP/2 以满足任何希望升级到 HTTP/2 的 HTTP/1.1 请求。它还将接受以 `PRI*HTTP/2.0\r\nSM\r\n `开始的h2c直接连接。

### HTTP2 push
服务器推送(Server Push)是 HTTP/2 支持的一个新功能，可以为单个客户端请求并行发送多个响应。
写到这我发现我真正应该该笔记的是啥
不过之前的价值也不太大吧。
+ 如何用本地jspush
+ 如何用本地的index.html（首次是用sendfile那里，很简单，但是组合可能就成为障碍
+ 稍加比对会发现有get和getnow两种，前者是**创造**一个HTTP GET,虽然大部分都被Deprecated(指在4中就会发生变化推荐使用WebClient)，返回一个Request，后者返回的是Client

### h2c
用h2c而不是tls
但是很奇怪的在网上显示的version是http1.1，而用客户端收到的就是2.0.【行吧客户端自己设了一下才收的数据，还说是不适用于不支持h2c的浏览器..嗯行吧，而且这应该就是正统h2c的用法了。

### custom frame
还以为frame是框架啥的。。发现就是帧，3个参数8bit的type，8bit的flags，后面就是payload

## Event Bus
### 点对点
两边都注册个函数，收发就行了。
+ 这里是用的一致名称来搜索地址的

### 发布/订阅
区别太细微以至于我一开始没看到。。。
+ 发布是publish，意味着都能收到。
+ 点对点是send，即便同地址有许多，也只会用不严格的轮询算法选择其中一个。

### 成了，Codecs
为了支持发送所有对象，你要为对象类型注册一个编解码器
问题是，API里找不到这个编解码器
当我想自行解决问题的时候，问题就变得无法解决了起来
成了，解决了。怪不得没有，原来注册编解码器的所有都是自己写的——这个CustomMessage根本就是在目录下的一个java文件而已
那这么说这个例子还挺有借鉴价值的
+ 编解码器要怎么写：实现哪些方法，哪些是固定格式，比如怎么出json
+ 