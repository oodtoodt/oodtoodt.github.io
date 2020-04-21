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
+ 整个流程
以下是胡比比：
sender先计时器用eventbus每秒向cluster receiver发一个自定义消息，等待reply
还部署了一个local receiver用的verticle实例，每两秒向local receiver发送消息并等待回复。

这里面的local receiver是自己写的，里面就是一个叫`local-message-receiver`的eventbus用来接收消息，可以认为是用verticle deploy了实例之后他就接进来了。
不不不。我觉得只是随便开了一个verticle，为了辨认起了local receiver的class的名字，目的就是异步，就是说我不在同一个eventloop里面搞多个事件。
我又变卦了。我现在觉得是前者。原因是创建verticle必然会把它分配给一个eventloop，并在里面执行它的start（极大可能是在当前eventloop里执行）
这里我理解就是，不需要你自己开这边这个了，另一个clusterreceiver还是需要自己开。local会自己开，甚至存在非常奇怪的继承

注意到在sender里面刚开始就注册了codec，后面就不用了。即便是在sender里面deploy的localreceiver内部的start，也不注册了。
这个继承关系让我摸不到头脑。
你要说这不是继承，那这是什么呢
某种神奇的注册机制，相当于把代码提上来？
反射？

经过半小时的纠缠，我现在看法又改变了。
我现在觉得是一个依赖线程或者说是eventloop的context在搞鬼。比如我们的context信息如果是相同的，草，我要做实验

实验做完了，debug一点一点看的。
首先，这个事就是个全局环境。两者的vertx是相同的——vertx是什么？vertx就是个全局的超级对象，是vert.x框架下的核心。而只要vertx相同，那么eventbus就相同。
而我是通过deploy的方式在当前的vertx里新建了verticle，所以我们的vertx是相同的，甚至上下文context也有非常多相同的：owner(就是vertx本尊了)，tccl（我们暂且认为是上下文装载器，某种filter），workerpoll和internal版本，orderedtasks和internal版本。
而如果！我新跑一个ClusterReceiver，那么这个vertx将和之前不同。这一不同几乎就是完！全！不！同！基本上找不到相同的数据了。

其他的没啥。基本上的逻辑就这样了。codec的内容我们以后再看...下一个下一个

### SSL
嗯so easy，就跟net的时候。。。
等等？这也太干脆了，就直接在点对点的基础上在调用时候使用
```java
Runner.runClusteredExample(Sender.class,
        new VertxOptions().setEventBusOptions(new EventBusOptions()
            .setSsl(true)
            .setKeyStoreOptions(new JksOptions().setPath("keystore.jks").setPassword("wibble"))
            .setTrustStoreOptions(new JksOptions().setPath("keystore.jks").setPassword("wibble"))
        )
    );
```
下面的start一模一样！！作者应该也是就这么复制的hhh
+ 如何传递ssl的eventbus,这里建议去看runner源码，下面会有解析

## Future
这东西是用来做异步结果协调的。主要的问题是，文档里没得解释...只能自己看了
教你写future/promise
>promise表示可能已经或可能尚未发生的动作的可写面(writable side)。可以用future（）方法返回与承诺相关联的Future，future可用于获取承诺完成的通知并检索其值。
>future表示可能已经或可能尚未发生的操作的结果(result)。

里面也写了mimic，即模拟一个消耗时间的操作和另一个消耗时间的操作异步顺序链接（依次执行异步调用）
+ 如何异步协调

## verticle
### deploy
这个结果太绝了
```
Main verticle has started, let's deploy some others...
In OtherVerticle.start
In OtherVerticle.start
In OtherVerticle.start
In OtherVerticle.start
In OtherVerticle.start
In OtherVerticle.start
Config is {}
Config is {}
Config is {}
Config is {}
Config is {}
Config is {"foo":"bar"}
Other verticle deployed ok, deploymentID = 58929df6-6dd1-4972-82e2-9be041e6c313
In OtherVerticle.stop
Undeployed ok!
```
嗯。那么首先start和config只是分了两个system.out.println而已，然后被deploy直接就分离了。
还有other...那个就是deploy的第二个参数而已，也接到非常神奇的后面了。
最后unploy倒是没什么问题，算是顺序的了。
值得一提的是带config的跑到最后了，估计就是谁跑得快谁开吧。
我们看下文档：（按顺序）

    + Deploying without waiting for it to deploy
    + Deploying and waiting for it to deploy(指要传参给res)
    + Passing configuration to another verticle during deploy(最慢)
    + Deploying more than one instance
    + Deploying as a worker verticle
    + Undeploying a verticle deployment explicitly

+ deploy的顺序
+ 理解异步
### 异步启动和终止
这里的参数就是`Future<Void> starFuture`了，见具体章节。
即需要时间来启动或者清理的时候，我们需要用这种启动来避免阻塞
不是问题
+ 异步启动

### worker verticle实例
```java
vertx.eventBus().<String>consumer(...)
```
太绝了，吓坏孩子了。
反正就是定义这个eventbus里面的`public <T>messageconsumer<T>`都是string了(T直接就是string了)
重点不在这！
重点在这里向我们展示了如何使用worker verticle，虽然只是配置了一下options，但是你会发现，虽然你开的worker是在0里开始的，但是！它consume eventbus的时候就是从worker-thread-1开始干的了
！！！
注意！你开的worker是从worker-0开始的，你的主verticle是从eventloop-thread-0开始的！！
我日，我改了半天想探究一下为啥俩0一个1，原来如此。
然后我还搞明白了一件事
这是异步！！！
我往**deploy的res里面**向外部的arraylist或者数组加东西，然后从**外面**undeploy这个获取到的id。我为啥反应这么慢呢，是因为我确实对java不熟，不能确认lambda里能否对外面的值进行改动（可以的）
想清楚了来龙去脉随便设计个timer就懂了。在外面println的时候a[0]依然是null或是原设值，在timer里等1ms(最低了，0都不让设的)就已经是改过的值了。(吹一波自己！periodic用timer解可还行，不过还是依赖了lambda传effective final的特性)
时刻记住这里到处都是异步。可能有些东西就想通了。
要是昨天先看这个例子我估计我就搞明白昨天那个sender里面开verticle是怎么回事了。
+ 理解异步
+ 如何用worker-verticle写多线程

## execblocking
指以不阻塞事件循环的方式来包含阻塞与非阻塞代码。
对runner产生了一点小小的想法。一会解析一下runner
这里又出现了`vertx.<String>executeBlocking`，但是你会发现，如果删去对于promise(第一个函参)是没影响的，但是对第二个函参res就有影响了，`res.result`变成了object编译错误。回过头看刚才那个例子，明白是影响到了后面的函参提供的构造(body())。
好，不扯闲篇，

+ 如何用executeblocking API来执行阻塞代码并获得阻塞代码执行后的异步回调的结果

这个结果看不太出什么，我们如果让他sleep5s，然后我们打开localhost8080的时候会被疯狂警告：
```
4月 21, 2020 2:16:29 下午 io.vertx.core.impl.BlockedThreadChecker
警告: Thread Thread[dedicated-pool-0,5,main]=Thread[dedicated-pool-0,5,main] has been blocked for 4887 ms, time limit is 0 ms
```
每秒一次。代表警告这个线程被阻塞的时间太久。
其他没啥说的，executeBlocking两个函参，第一个是要阻塞的代码是什么，第二个是异步回调执行结果

## HA
这里是展示高可用性的，当原始节点死亡时，会将顶点重新部署到另一个节点
不清楚ide里能不能模拟集群环境
初步失败了，感觉不知道怎么杀了这个进程，kill找不到progress，wsl又蠢得一比


## JavaScript Verticle and NPM
没啥关系不看了。

## json
+ 如何用流json来解析包含许多小对象的大数组。
+ mapTo的Json器怎么写
唔，依赖com.fasterxml.jackson这个东西。pom有点问题，一层层找上去发现是依赖崩了。亏我还去查了半天的文档。接受教训，先看依赖，再找文档

是在用ReadStream的接口，用AsyncFile来读写实现这个接口的。
注意到Http|C/S|Request、MessageConsumer、Net/Web|Socket都实现了这个接口的
流(完全)到达时调用endHandler。
handler：设置一个处理器，它将从ReadStream读取项
没什么好看的了