---
title: vert.x-notes
date: 2020-04-15 10:29:43
tags:
- java
- company

categories:
- java
- frame
- vert.x
- documentation

---

我...我把整篇文章搬过来了！
复制粘贴...
https://vertxchina.github.io/vertx-translation-chinese/core/Core.html
https://vertx.io/docs/vertx-core/java/#_buffers
<!--more-->



## Are you fluent
A fluent API是可以连锁调用许多方法的,比如
```java
request.response().puHeader("Content-Type","text/plain").write("some text").end();
//but we dont force you, you still can ignore it
HttpServerResponse response = request.response();
response.putHeader("Content-Type","text/plain");
response.write("some text");
response.end();
```

## Don't call us, we'll call you
Vert.x APIs主要是事件驱动的。意味着如果Vert.x中发生的事情你感兴趣，Vert.x会通过事件的方法调用你

>Handler主要用于异步消息的处理（是一套Android提供的消息处理机制）：当发出一个消息之后，首先进入一个消息队列，发送消息的函数即刻返回，而另外一个部分在消息队列中逐一将消息取出，然后对消息进行处理，也就是发送消息和接收消息不是同步的处理。 这种机制通常用来处理相对耗时比较长的操作。
（无视就好，和这个什么关系都没有

这里的Lambda/匿名函数/req->{//blablabla}就是一个处理器（Handler），在随后的例子中，我们用1stHandler以及2ndHandler来指代具体的匿名函数，让代码更加清晰明了，放在Verticle中类似：

```java
public class MyVerticle extends AbstractVerticle {
    public void start() throws Exception {
        vertx.createHttpServer().requestHandler(1stHandler).listen(8080);
    vertx.createHttpServer().requestHandler(2ndHandler).listen(8081);
    }
}
```

你通过提供handlers来handle事件，比如如果你要收到一个时间事件你应该：
```java
vertx.setPeriodic(1000, id -> { // This handler will get called every second System.out.println("timer fired!");
});
//HTTP request
server.requestHandler(request -> {
  // This handler will be called every time an HTTP request is received at the server
  request.response().end("hello world!");
});
```
过以后当Vert.x有一个事件经过你的handler时，Vert.x将异步调用它。
这使我们想到Vert.x中的一些重要概念

## Don't block me!
除了极少数例外（某些文件系统以'Sync'结尾）,Vert.x中所有API均不会阻塞调用线程
如果可以立即提供结果，将立即返回结果，否则通常会在一段时间后提供处理程序以接受事件
没有Vert.xAPI阻塞意味着你可以用少量线程来处理大量并发
传统的blocking API可能会在以下情况下block：
+ 读数据from a socket
+ 写数据到磁盘
+ 向收件人发送消息并等待回复
+ 许多其他情况
上述情况下您的线程等待结果时无法执行其他任何操作——就很没用
阻塞API进行大量并发操作需要大量线程防止应用程序停止运行，线程在其所需内存（堆栈）和上下文切换方面都有开销
所以在现代应用所需的并发级别中，blocking方法无法扩展

## Reactor and Multi-Reactor
在大多数情况下，Vert.x使用称为事件循环(event loop)的线程来调用处理程序。
「由于没有blocks」这句话将会大量的看到。因为它，event loop可以轻松地在事件到达时连续地将事件传递给不同的处理程序。event loop可以潜在地在短时间内交付大量事件，比如单event loop可以快速处理数千个HTTP requests——
我们称之为 Reactor Pattern（反应堆模式）
Node.js也实现了这种模式，但这不太一样。
>在标准的反应堆实现中，只有一个事件循环线程(single event loop thread)，该线程在一个循环中运行，将所有事件到达时将所有事件传递给它们。
>单线程的麻烦是只能在任何时间运行在单个内核上，因此，如果您希望单线程反应堆应用程序能够在多核服务器上扩展，则必须启动并管理许多不同的流程
Vert.x在这里的工作方式有所不同。每个Vertx实例都维护多个事件循环，而不是单个事件循环。默认情况下，我们根据计算机上可用内核的数量选择数量，可以选择覆写
意味着单个Vertx进程可以在整个服务器上扩展
我们称之为多反应堆模式
即使Vertx实例维护多个事件循环，也不会同步执行任何特定handler，绝大数时间将使用完全相同的event loop进行调用

## The Golden Rule - Don't Block the Event Loop
不要block event loop **yourself**！包括：
+ thread.sleep()
+ Waiting on a lock（等待锁）
+ Waiting on a mutex or monitor(等待互斥或监视器 比如synchronized section）
+ 进行长时间的数据库操作并等待结果
+ 进行复杂的需要花费大量时间的计算
+ 自旋
那么多久的时间是可以等待的呢？取决于你的程序和并发量
**数学并不难。**
如果您的应用程序没有响应，可能表明您在某处阻止了event loop，Vertx会自动记录警告，如果您在日志中看到这些警告您应进行调查：
```
Thread vertx-eventloop-thread-3 has been blocked for 20458 ms
```
Vertx还将提供堆栈跟踪来精确定位
在创建Vertx对象之前改变VertxOptions对象可以改变这些设置。

## Running blocking code
很多库，尤其是JVM生态系统中，有同步APIs和大量阻塞方法。像JDBC API，它固有的同步，那么无论Vertx多么努力，都无法使其变成异步的。
我们不会在一夜之间将所有方法写成异步的，所以我们提供了在Vertx安全使用传统的blocking APIs的办法
通过调用`executeBlocking`同时指定要执行的阻塞代码和执行了阻塞代码后被异步回调的结果程序来完成这个任务
```java
vertx.excuteBlocking(promise ->{
  //Call some blocking API that takes a significant amount of time to return
  String result = someAPI.blockingMethod("hello");
  promise.complete(result);
},res ->{
  System.out.println("The result is: " + res.result());
})
```
>阻塞代码应在合理的时间内阻塞，这排除了长阻塞和轮询操作。当阻塞10秒钟以上时，阻塞线程检查器将在控制台上显示一条消息。长阻塞操作应使用由应用程序管理的专用线程，可以和事件总线或`runOnContext`和verticles交互。

如果从同一上下文（比如相同的verticle 实例）多次调用`executeBlocking`，则会依次执行不同的`executeBlocking`.
如果您不关心顺序，那么`executeBlocking`也可以特定为ordered为false，任何executeBlocking都可能在工作池上并行执行
另一种可以选择的方法是用工作verticle(worker verticle)
Worker verticle总是通过一个worker pool里的来的线程执行的
blocking code默认在Vertx worker pool上执行，配置于`setWorkderPoolSize`
可以出于其他目的创造pools：
```java
WorkerExecutor executor = vertx.createSharedWorkerExecutor("my-worker-pool");
executor.executeBlocking(promise -> {
  // Call some blocking API that takes a significant amount of time to return
  String result = someAPI.blockingMethod("hello");
  promise.complete(result);
}, res -> {
  System.out.println("The result is: " + res.result());
});
```
注意worker executor如果不再用必须被关上
```java
executor.close();
```
当使用相同的名称创建多个workers，它们会享用相同的pool，worker pool在所有worker executors关闭时会被销毁。
一个executor在Verticle创建后，Vertx会自动关闭它当Verticle取消部署后
可以在创建executors时配置它
```java
int poolSize = 10;

// 2 minutes
long maxExecuteTime = 2;
TimeUnit maxExecuteTimeUnit = TimeUnit.MINUTES;

WorkerExecutor executor = vertx.createSharedWorkerExecutor("my-worker-pool", poolSize, maxExecuteTime, maxExecuteTimeUnit);
```

## Async coordination（异步协调）
Vertx `futures`可以实现多个异步结果的协调。它支持并发组合（并行运行多个异步操作）和顺序组合（链异步操作）

### Concurrent composition(并发组成[合并?])
`CompositeFuture.all`接受几个futures参数（最多6个），并返回一个future，当所有future都成功时，该future将成功；如果至少一个future失败，则将失败
```java
Future<HttpServer> httpServerFuture = Future.future(promise -> httpServer.listen(promise));

Future<NetServer> netServerFuture = Future.future(promise -> netServer.listen(promise));

CompositeFuture.all(httpServerFuture, netServerFuture).onComplete(ar -> {
  if (ar.succeeded()) {
    // All servers started
  } else {
    // At least one server failed
  }
});
```
所有被合并的` Future `中的操作同时运行。当组合的处理操作完成时，该方法返回的` Future `上绑定的处理器（Handler）会被调用。可以传入一个`Future`列表（可能为空）
```java
CompositeFuture.all(Arrays.asList(future1, future2, future3));
```

any方法会等待第一个成功执行的Future，当任意一个Future成功得到结果，则该`Future`成功。
```java
CompositeFuture.any(future1, future2).onComplete(ar -> {
  if (ar.succeeded()) {
    // At least one is succeeded
  } else {
    // All failed
  }
});
```

`join`方法的合并会等待所有的Future完成，无论成败，并将结果归并成一个Future，当至少一个Future执行失败，得到的Future就是失败的
```java
CompositeFuture.join(future1, future2, future3).onComplete(ar -> {
  if (ar.succeeded()) {
    // All succeeded
  } else {
    // All completed and at least one failed
  }
});
```
都可以传列表参，都最多有6个的限制。

### Sequential composition（顺序合并）
与`all`和`any`实现的并发组合不同，`compose`方法用于顺序组合`Future`（chaining futures)
```java
FileSystem fs = vertx.fileSystem();

Future<Void> fut1 = Future.future(promise -> fs.createFile("/foo", promise));

Future<Void> startFuture = fut1
  .compose(v -> {
  // When the file is created (fut1), execute this:
  return Future.<Void>future(promise -> fs.writeFile("/foo", Buffer.buffer(), promise));
})
  .compose(v -> {
  // When the file is written (fut2), execute this:
  return Future.future(promise -> fs.move("/foo", "/bar", promise));
});
```
1. 一个文件被创建(fut1)
2. 一些东西被写入到文件(fut2)
3. 文件被移走(startFuture)
如果全部成功，那么最终的startFuture也是成功的。

例子中使用了：
+ `compose(mapper)`:当前future完成时，执行给出的函数体，返回一个future，返回的future完成时，组合完成
+ `compose(handler,next)`:当前future，执行相关代码并完成下一个future的处理（给出的处理器完成了给出的`next` future)
第二个例子中，处理器需要完成`next` future，以此来汇报处理成功或者失败

## Verticles
Vert.x通过开箱即用的方式提供了一个简单便捷的、可扩展的、类似Actor Model的部署和并发模型机制。您可以用此模型机制来保管您自己的代码组件。
它是可选的。
>Verticle 是由 Vert.x 部署和运行的代码块。默认情况一个 Vert.x 实例维护了N（默认情况下N = CPU核数 x 2）个 Event Loop 线程。Verticle 实例可使用任意 Vert.x 支持的编程语言编写，而且一个简单的应用程序也可以包含多种语言编写的 Verticle。

这个模型不能说是严格的Actor模式的实现，但它确实有相似之处，特别是在并发、扩展性和部署等方面
您可以将 Verticle 想成 Actor Model 中的 Actor。
要使用该模型，您需要将您的代码组织成一系列的**Verticle**。

一个应用程序通常是由在同一个 Vert.x 实例中同时运行的许多 Verticle 实例组合而成。不同的 Verticle 实例通过向 Event Bus 上发送消息来相互通信。

### Writing Verticle
必须实现Verticle接口
直接从抽象类AbstractVerticle继承更简单。
```java
public class MyVerticle extends AbstractVerticle {

 // Called when verticle is deployed
 public void start() {
 }

 // Optional - called when verticle is undeployed
 public void stop() {
 }

}
```
通常您需要重写start。Vertx部署Verticle时，它的start方法将被调用。
stop同理，Vertx撤销一个Verticle时它会被调用

### Asynchronous Verticle start and stop(异步启动和终止)
比如您想在Verticle启动耗费时间的这个过程中做一些事，比如在`start`方法中部署其他的Verticle——
您不能在start方法中阻塞等待其他的Verticle部署完成
您可以实现异步版本的start来做它。这个版本的方法会以一个 Future 作参数被调用。方法执行完时，Verticle 实例**并没有**部署好（状态不被认为 deployed）。
稍后，您完成了所有您需要做的事（如：启动其他Verticle），您就可以调用 Future 的 complete（或 fail ）方法来标记启动完成或失败了
```java
public class MyVerticle extends AbstractVerticle {

 private HttpServer server;

 public void start(Future<Void> startFuture) {
   server = vertx.createHttpServer().requestHandler(req -> {
     req.response()
       .putHeader("content-type", "text/plain")
       .end("Hello from Vert.x!");
     });

   // Now bind the server:
   server.listen(8080, res -> {
     if (res.succeeded()) {
       startFuture.complete();
     } else {
       startFuture.fail(res.cause());
     }
   });
 }
}
//same stop method too
public class MyVerticle extends AbstractVerticle {

 public void start() {
   // Do something
 }

 public void stop(Future<Void> stopFuture) {
   obj.doSomethingThatTakesTime(res -> {
     if (res.succeeded()) {
       stopFuture.complete();
     } else {
       stopFuture.fail();
     }
   });
 }
}
```
不需要手动stop verticle启动的 HTTP server，会在...的时候自动撤销

### Verticle types
+ Stardand Verticle：这是最常用的一类 Verticle —— 它们永远运行在 Event Loop 线程上。稍后的章节我们会讨论更多。
+ Worker Verticle：这类 Verticle 会运行在 Worker Pool 中的线程上。一个实例绝对不会被多个线程同时执行。

#### Standard Verticle
当 Standard Verticle 被创建时，它会被分派给一个 Event Loop 线程，并在这个 Event Loop 中执行它的 start 方法。当您在一个 Event Loop 上调用了 Core API 中的方法并传入了处理器时，Vert.x 将保证用与调用该方法时相同的 Event Loop 来执行这些处理器。

这意味着我们可以保证您的 Verticle 实例中所有的代码都是在相同Event Loop中执行（只要您不创建自己的线程并调用它！）

同样意味着您可以将您的应用中的所有代码用单线程方式编写，让 Vert.x 去考虑线程和扩展问题。您不用再考虑 synchronized 和 volatile 的问题，也可以避免传统的多线程应用经常会遇到的竞态条件和死锁的问题。

#### worker Verticle
正如之前所说的，这东西被设计来调用阻塞式代码，不会阻塞Event loop，由Vert.x中的 Worker Pool 中的线程执行。
如果您不想使用 Worker Verticle 来运行阻塞式代码，您还可以在一个Event Loop中直接使用 内联阻塞式代码。
若您想要将 Verticle 部署成一个 Worker Verticle，您可以通过 setWorker 方法来设置：
```java
DeploymentOptions options = new DeploymentOptions().setWorker(true);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```
Worker Verticle 实例绝对不会在 Vert.x 中被多个线程同时执行，但它可以在不同时间由不同线程执行。


>警告：Multi-threaded Worker Verticle 是一个高级功能，大部分应用程序不会需要它。

#### Deploying verticles programmatically
>通过 Verticle 实例 来部署 Verticle 仅限Java
您可以指定一个 Verticle 名称或传入您已经创建好的 Verticle 实例，使用任意一个 deployVerticle 方法来部署Verticle。
```java
Verticle myVerticle = new MyVerticle();
vertx.deployVerticle(myVerticle);
```
指定 Verticle 的 名称 来部署它，这个名称会用于查找实例化 Verticle 的特定 VerticleFactory。
不同的 Verticle Factory 可用于实例化不同语言的 Verticle，也可用于其他目的，例如加载服务、运行时从Maven中获取Verticle实例等。
```java
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle");

// 部署JavaScript的Verticle
vertx.deployVerticle("verticles/myverticle.js");

// 部署Ruby的Verticle
vertx.deployVerticle("verticles/my_verticle.rb");
```

#### Rules for mapping a verticle name to a verticle factory(Verticle名称到Factory的映射规则)
Verticle 名称可以有一个前缀 —— 使用字符串紧跟着一个冒号，它用于查找存在的Factory，参考：
```java
js:foo.js // Use the JavaScript verticle factory
groovy:com.mycompany.SomeGroovyCompiledVerticle // Use the Groovy verticle factory 
service:com.mycompany:myorderservice // Uses the service verticle factory
```
如果不指定前缀，Vert.x将根据提供名字后缀来查找对应Factory，如：
```java
foo.js // 将使用JavaScript的Factory
SomeScript.groovy // 将使用Groovy的Factory
```
都没指定，Vert.x将假定这个名字是一个Java 全限定类名（FQCN）然后尝试实例化它。

#### How are Verticle Factories located?
大部分Verticle Factory会从 classpath 中加载，并在 Vert.x 启动时注册。

您同样可以使用编程的方式去注册或注销Verticle Factory：通过`registerVerticleFactory`方法和`unregisterVerticleFactory`方法。

#### Waiting for deployment to complete
Verticle的部署是异步方式，可能在`deploy`方法调用返回后一段时间才会完成部署。

如果您想要在部署完成时被通知则可以指定一个完成处理器：
```java
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", res -> {
  if (res.succeeded()) {
    System.out.println("Deployment id is: " + res.result());
  } else {
    System.out.println("Deployment failed!");
  }
});
```
如果部署成功，这个完成处理器的结果中将会包含部署ID的字符串。这个部署 ID可以在之后您想要撤销它时使用。

#### Undeploying verticle deployments
我们可以通过`undeploy`方法来撤销部署好的 Verticle。

撤销操作也是异步的，因此若您想要在撤销完成过后收到通知则可以指定另一个完成处理器：
```java
vertx.undeploy(deploymentID, res -> {
  if (res.succeeded()) {
    System.out.println("Undeployed ok");
  } else {
    System.out.println("Undeploy failed!");
  }
});
```

#### Specifying number of veritcle instances(设置 Verticle 实例数)
当使用名称部署一个 Verticle 时，您可以指定您想要部署的 Verticle 实例的数量。
```java
DeploymentOptions options = new DeploymentOptions().setInstances(16);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```
这个功能对于跨多核扩展时很有用。例如，您有一个实现了Web服务器的Verticle需要部署在多核的机器上，您可以部署多个实例来利用所有的核。

#### Passing configuration to a verticle
可在部署时传给 Verticle 一个 JSON 格式的配置
```java
JsonObject config = new JsonObject().put("name", "tim").put("directory", "/blah");
DeploymentOptions options = new DeploymentOptions().setConfig(config);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```
传入之后，这个配置可以通过 Context 对象或使用 config 方法访问。

这个配置会以 JSON 对象（JsonObject）的形式返回，因此您可以用下边代码读取数据：
```java
System.out.println("Configuration: " + config().getString("name"));
```

#### Accessing environment variables in a Verticle(在 Verticle 中访问环境变量)
环境变量和系统属性可以直接通过 Java API 访问：

System.getProperty("prop");
System.getenv("HOME");

#### Verticle Isolation Groups
>警告：谨慎使用此功能，类加载器可能会导致您的应用难于调试，变得一团乱麻（can of worms）。
您可能需要部署一个Verticle，它包含的类要与应用程序中其他类隔离开来。比如您想要在一个Vert.x实例中部署两个同名不同版本的Verticle，或者不同的Verticle使用了同一个jar包的不同版本。
当使用隔离组时，您需要用 setIsolatedClassed 方法来提供一个您想隔离的类名列表。列表项可以是一个Java 限定类全名，也可以是包含通配符的可匹配某个包或子包的任何类
```java
DeploymentOptions options = new DeploymentOptions().setIsolationGroup("mygroup");
options.setIsolatedClasses(Arrays.asList("com.mycompany.myverticle.*",
                   "com.mycompany.somepkg.SomeClass", "org.somelibrary.*"));
vertx.deployVerticle("com.mycompany.myverticle.VerticleClass", options);
```

#### High Availability
在这种方式下，当其中一个部署在 Vert.x 实例中的 Verticle 突然挂掉，这个 Verticle 可以在集群环境中的另一个 Vert.x 实例中重新部署。
若要启用高可用方式运行一个 Verticle，仅需要追加 -ha 参数，且无需 -cluster：
`vertx run my-verticle.js -ha`

#### Running Verticles from the command line
您可以在 Maven 或 Gradle 项目中以正常方式添加 Vert.x Core 为依赖，在项目中直接使用 Vert.x。

#### Causing Vert.x to exit
Vert.x 实例维护的线程不是守护线程，因此它们会阻止JVM退出。

如果您通过嵌入式的方式使用 Vert.x 并且完成了操作，您可以调用 close 方法关闭它。这将关闭所有内部线程池并关闭其他资源，允许JVM退出。

#### The Context object
每个 Verticle 在部署的时候都会被分配一个 Context（根据配置不同，可以是Event Loop Context 或者 Worker Context），之后此 Verticle 上所有的普通代码都会在此 Context 上执行（即对应的 Event Loop 或Worker 线程）
您可以通过 getOrCreateContext 方法获取 Context 实例
若已经有一个 Context 和当前线程关联，那么它直接重用这个 Context 对象，如果没有则创建一个新的。您可以检查获取的 Context 的类型：
```java
Context context = vertx.getOrCreateContext();
if (context.isEventLoopContext()) {
  System.out.println("Context attached to Event Loop");
} else if (context.isWorkerContext()) {
  System.out.println("Context attached to Worker Thread");
} else if (context.isMultiThreadedWorkerContext()) {
  System.out.println("Context attached to Worker Thread - multi threaded worker");
} else if (! Context.isOnVertxThread()) {
  System.out.println("Context not attached to a thread managed by vert.x");
}
```
当您获取了这个 Context 对象，您就可以在 Context 中异步执行代码了。换句话说，您提交的任务将会在同一个 Context 中运行：
```java
vertx.getOrCreateContext().runOnContext( (v) -> {
  System.out.println("This will be executed asynchronously in the same context");
});
```
当在同一个 Context 中运行了多个处理函数时，可能需要在它们之间共享数据。 Context 对象提供了存储和读取共享数据的方法。举例来说，它允许您将数据传递到 runOnContext 方法运行的某些操作中：
```java
final Context context = vertx.getOrCreateContext();
context.put("data", "hello");
context.runOnContext((v) -> {
  String hello = context.get("data");
});
```

#### Executing periodic and delayed actions(执行周期性/延迟性操作)
在 Vert.x 中，想要延迟之后执行或定期执行操作很常见。

在 Standard Verticle 中您不能直接让线程休眠以引入延迟，因为它会阻塞 Event Loop 线程。取而代之是使用 Vert.x 定时器。定时器可以是一次性或周期性的

##### one-shot Timers
一次性计时器会在一定延迟后调用一个 Event Handler，以毫秒为单位计时。
您可以通过`setTimer`方法传递延迟时间和一个处理器来设置计时器的触发。
返回值是一个唯一的计时器id，该id可用于之后取消该计时器，这个计时器id会传入给处理器。
```java
long timerID = vertx.setTimer(1000, id -> {
  System.out.println("And one second later this is printed");
});

System.out.println("First this is printed");
```

##### Periodic Timers
和一次性的一模一样，只不过...
请记住这个计时器将会定期触发。如果您的定时任务会花费大量的时间，则您的计时器事件可能会连续执行甚至发生更坏的情况：重叠。这种情况，您应考虑使用`setTimer`方法，当任务执行完成时设置下一个计时器。
```java
long timerID = vertx.setPeriodic(1000, id -> {
  System.out.println("And every second this is printed");
});

System.out.println("First this is printed");
```

##### 取消、清除
指定id,cancelTimer
```java
vertx.cancelTimer(timerID);
```

清除依然是Verticle被撤销时自动关闭

#### Verticle worker pool
Verticle 使用 Vert.x 中的 Worker Pool 来执行阻塞式行为，例如 executeBlocking 或 Worker Verticle。

可以在部署配置项中指定不同的Worker 线程池：
```java
vertx.deployVerticle("the-verticle", new DeploymentOptions().setWorkerPoolName("the-specific-pool"));
```

### The Event Bus
Event Bus 是 Vert.x 的**神经系统**。

每一个 Vert.x 实例都有一个单独的 Event Bus 实例。您可以通过 Vertx 实例的`eventBus`方法来获得对应的EventBus实例。

您的应用中的不同部分通过 Event Bus 相互通信，无论它们使用哪一种语言实现，无论它们在同一个 Vert.x 实例中或在不同的 Vert.x 实例中。

甚至可以通过桥接的方式允许在浏览器中运行的客户端JavaScript在相同的Event Bus上相互通信。

Event Bus可形成跨越多个服务器节点和多个浏览器的点对点的分布式消息系统。

Event Bus支持发布/订阅、点对点、请求/响应的消息通信方式。

Event Bus的API很简单。基本上只涉及注册处理器、撤销处理器和发送和发布消息。

首先来看些基本概念和理论。

#### The Theory

##### Addressing
消息会被 Event Bus 发送到一个 地址(address)。

同任何花哨的寻址方案相比，Vert.x的地址格式并不麻烦。Vert.x中的地址是一个简单的字符串，任意字符串都合法。当然，使用某种模式来命名仍然是明智的。如：使用点号来划分命名空间。

一些合法的地址形如：`europe.news.feed1`、`acme.games.pacman`、`sausages`和`X`。

##### Handlers
消息在处理器（Handler）中被接收。您可以在某个地址上注册一个处理器来接收消息。

同一个地址可以注册许多不同的处理器，一个处理器也可以注册在多个不同的地址上。

##### publish/subscribe messaging
发布/订阅消息
Event Bus支持**发布消息**功能。

消息将被发布到一个地址中，发布意味着会将信息传递给所有注册在该地址上的处理器。这和**发布/订阅模式**很类似。

##### Point-to-point and Request-Response messaging
Event Bus也支持**点对点**消息模式。

消息将被发送到一个地址中，Vert.x将会把消息分发到某个注册在该地址上的处理器。若这个地址上有不止一个注册过的处理器，它将使用 不严格的轮询算法 选择其中一个。

点对点消息传递模式下，可在消息发送的时候指定一个应答处理器（可选）。

当接收者收到消息并且已经被处理时，它可以选择性决定回复该消息，若选择回复则绑定的应答处理器将会被调用。当发送者收到回复消息时，它也可以回复，这个过程可以不断重复。通过这种方式可以允许在两个不同的 Verticle 之间设置一个对话窗口。这种消息模式被称作**请求-响应**模式。

##### Best-effort delivery
Vert.x会尽它最大努力去传递消息，并且不会主动丢弃消息。这种方式称为 尽力传输(Best-effort delivery)。

但是，当 Event Bus 中的全部或部分发生故障时，则可能会丢失消息。

若您的应用关心丢失的消息，您应该编写具有幂等性的处理器，并且您的发送者可以在恢复后重试。(接口幂等：无论客户端调用服务端接口发起多少次请求，有且只有一次有效。)

RPC通信通常情况下有三种语义：at least once、at most once 和 exactly once。不同语义情况下要考虑的情况不同。

##### Types of messages
Vert.x 默认允许任何基本/简单类型、String 或`buffers`作为消息发送。不过在 Vert.x 中的通常做法是使用 JSON 格式来发送消息。

JSON 对于 Vert.x 支持的所有语言都是非常容易创建、读取和解析的，因此它已经成为了Vert.x中的通用语(lingua franca)。但是若您不想用 JSON，我们并不强制您使用它。

Event Bus 非常灵活，它支持在 Event Bus 中发送任意对象。您可以通过为您想要发送的对象自定义一个`codec`来实现。

#### The Event Bus API

##### Getting the event bus
```java
EventBus eb = vertx.eventBus();
```
对于每一个 Vert.x 实例来说它是单例的。

##### Registering Handlers
最简单的注册处理器的方式是使用 consumer 方法，这儿有个例子：
```java
EventBus eb = vertx.eventBus();

eb.consumer("news.uk.sport", message -> {
  System.out.println("I have received a message: " + message.body());
});
```
当一个消息达到您的处理器，该处理器会以 message 为参数被调用。

调用 consumer 方法会返回一个 MessageConsumer 对象。该对象随后可用于撤销处理器、或将处理器用作流式处理。

您也可以不设置处理器而使用 consumer 方法直接返回一个 MessageConsumer，之后再来设置处理器。如：
```java
EventBus eb = vertx.eventBus();

MessageConsumer<String> consumer = eb.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
});
```
在集群模式下的Event Bus上注册处理器时，注册信息会花费一些时间才能传播到集群中的所有节点。

若您希望在完成注册后收到通知，您可以在 MessageConsumer 对象上注册一个 completion handler。
```java
consumer.completionHandler(res -> {
  if (res.succeeded()) {
    System.out.println("The handler registration has reached all nodes");
  } else {
    System.out.println("Registration failed!");
  }
});
```

##### Un-registering Handlers
您可以通过 unregister() 方法来注销处理器。

若您在集群模式下的 Event Bus 中撤销处理器，则同样会花费一些时间在节点中传播。若您想在完成后收到通知，可以使用unregister(handler) 方法注册处理器：
```java
consumer.unregister(res -> {
  if (res.succeeded()) {
    System.out.println("The handler un-registration has reached all nodes");
  } else {
    System.out.println("Un-registration failed!");
  }
});
```

##### Publishing messages
发布消息很简单，只需使用`publish`方法指定一个地址去发布即可。
```java
eventBus.publish("news.uk.sport", "Yay! Someone kicked a ball");
```
这个消息将会传递给所有在地址 news.uk.sport 上注册过的处理器。

##### Sending messages
与发布消息的不同之处在于，发送(send)的消息只会传递给在该地址注册的其中一个处理器，这就是点对点模式。Vert.x 使用不严格的轮询算法来选择绑定的处理器。
publish改成send即可

##### Setting headers on messages
在 Event Bus 上发送的消息可包含头信息。这可通过在发送或发布时提供的 DeliveryOptions 来指定。例如：
```java
DeliveryOptions options = new DeliveryOptions();
options.addHeader("some-header", "some-value");
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball", options);
```

##### Message ordering
Vert.x将按照特定发送者发送消息的顺序来传递消息给特定处理器。

##### The Message object
您在消息处理器中接收到的对象的类型是`Message`。

消息的`body`对应发送或发布的对象。消息的头信息可以通过`headers`方法获取。

##### Acknowledging messages / sending replies
当使用 send 方法发送消息时，Event Bus会尝试将消息传递到注册在Event Bus上的 MessageConsumer中。在某些情况下，发送者需要知道消费者何时收到消息并*处理*了消息。

消费者可以通过调用`reply`方法来应答这个消息。

当这种情况发生时，它会将消息回复给发送者并且在发送者中调用应答处理器来处理回复的消息。
看这个例子会更清楚：
```java
//Thre receiver
MessageConsumer<String> consumer = eventBus.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
  message.reply("how interesting!");
});

//The sender
eventBus.request("news.uk.sport", "Yay! Someone kicked a ball across a patch of grass", ar -> {
  if (ar.succeeded()) {
    System.out.println("Received reply: " + ar.result().body());
  }
});
```

##### Sending with timeouts
当发送带有应答处理器的消息时，可以在 DeliveryOptions 中指定一个超时时间。如果在这个时间之内没有收到应答，则会以失败为参数调用应答处理器。默认超时是 30 秒。

##### Send Failures
消息发送可能会因为其他原因失败，包括：

+ 没有可用的处理器来接收消息
+ 接收者调用了 fail 方法显式声明失败

发生这些情况时，应答处理器将会以这些失败为参数被调用。

##### Message Codecs
您可以在 Event Bus 中发送任何对象，只要你为这个对象类型注册一个编解码器 MessageCodec。消息编解码器有一个名称，您需要在发送或发布消息时通过 DeliveryOptions 来指定：
```java
eventBus.registerCodec(myCodec);

DeliveryOptions options = new DeliveryOptions().setCodecName(myCodec.name());

eventBus.send("orders", new MyPOJO(), options);
```
若您总是希望某个类使用将特定的编解码器，那么您可以为这个类注册默认编解码器。这样您就不需要在每次发送的时候使用 DeliveryOptions 来指定了：
```java
eventBus.registerDefaultCodec(MyPOJO.class, myCodec);

eventBus.send("orders", new MyPOJO());
```
您可以通过 unregisterCodec 方法注销某个消息编解码器。

消息编解码器的编码和解码不一定使用同一个类型。例如您可以编写一个编解码器来发送 MyPOJO 类的对象，但是当消息发送给处理器后解码成 MyOtherPOJO 对象。

##### Clustered Event Bus
Event Bus 不仅仅存在于单个 Vert.x 实例中。通过您在网络上将不同的 Vert.x 实例集群在一起，它可以形成一个单一的、分布式的Event Bus。

##### Clustering programmatically
若您用编程的方式创建 Vert.x 实例（Vertx），则可以通过将 Vert.x 实例配置成集群模式来获取集群模式的Event Bus：
```java
VertxOptions options = new VertxOptions();
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
    EventBus eventBus = vertx.eventBus();
    System.out.println("We now have a clustered event bus: " + eventBus);
  } else {
    System.out.println("Failed: " + res.cause());
  }
});
```

##### Clustering on the command line
-cluster

#### Automatic clean-up in verticles
若您在 Verticle 中注册了 Event Bus 的处理器，那么这些处理器在 Verticle 被撤销的时候会自动被注销。

#### Configuring the event bus
Event Bus 是可以配置的，这对于以集群模式运行的 Event Bus 是非常有用的。Event Bus 使用 TCP 连接发送和接收消息，因此可以通过`EventBusOptions`对TCP连接进行全面的配置。由于 Event Bus 同时用作客户端和服务器，因此这些配置近似于`NetClientOptions`和`NetServerOptions`。
```java
VertxOptions options = new VertxOptions()
    .setEventBusOptions(new EventBusOptions()
        .setSsl(true)
        .setKeyStoreOptions(new JksOptions().setPath("keystore.jks").setPassword("wibble"))
        .setTrustStoreOptions(new JksOptions().setPath("keystore.jks").setPassword("wibble"))
        .setClientAuth(ClientAuth.REQUIRED)
    );

Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
    EventBus eventBus = vertx.eventBus();
    System.out.println("We now have a clustered event bus: " + eventBus);
  } else {
    System.out.println("Failed: " + res.cause());
  }
});
```

上边代码段描述了如何在Event Bus中使用SSL连接替换传统的TCP连接。

>警告：若要在集群模式下保证安全性，您 必须 将集群管理器配置成加密的或强制安全的。参考集群管理器的文档获取更多细节。

Event Bus 的配置需要在所有集群节点中保持一致性。

EventBusOptions还允许您指定 Event Bus 是否运行在集群模式下，以及它的主机信息和端口。您可使用 setClustered、getClusterHost和 getClusterPort 方法来设置。

在容器中使用时，您也可以配置公共主机和端口号：
```java
VertxOptions options = new VertxOptions()
    .setEventBusOptions(new EventBusOptions()
        .setClusterPublicHost("whatever")
        .setClusterPublicPort(1234)
    );

Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
    EventBus eventBus = vertx.eventBus();
    System.out.println("We now have a clustered event bus: " + eventBus);
  } else {
    System.out.println("Failed: " + res.cause());
  }
});
```

## JSON
和其他一些语言不同，Java 没有对 JSON 的原生支持（first class support），因此我们提供了两个类，以便在 Vert.x 应用中处理 JSON 更容易。

### JSON object
JsonObject 类用来描述JSON对象。

一个JSON 对象基本上只是一个 Map 结构。它具有字符串的键，值可以是任意一种JSON 支持的类型（如 string, number, boolean）。

JSON 对象也支持 null 值。

#### Creating JSON objects 创建 JSON 对象
可以使用默认构造函数创建空的JSON对象。

您可以通过一个 JSON 格式的字符串创建JSON对象：
```java
String jsonString = "{\"foo\":\"bar\"}";
JsonObject object = new JsonObject(jsonString);
```
您可以从通过一个Map创建JSON对象：
```java
Map<String, Object> map = new HashMap<>();
map.put("foo", "bar");
map.put("xyz", 3);
JsonObject object = new JsonObject(map);
```

#### Putting entries into a JSON object 将键值对放入 JSON 对象
使用put 方法可以将值放入到JSON对象里。

这个API是流式的，因此这个方法可以被链式地调用。
```java
JsonObject object = new JsonObject();
object.put("foo", "bar").put("num", 123).put("mybool", true);
```

#### Getting values from a JSON object 从 JSON 对象获取值
您可使用 getXXX 方法从JSON对象中获取值。例如：
```java
String val = jsonObject.getString("some-key");
int intVal = jsonObject.getInteger("some-other-key");
```

#### Mapping between JSON objects and Java objects JSON 对象和 Java 对象间的映射
您可以从 Java 对象的字段创建一个JSON 对象，如下所示：

你可以通过一个JSON 对象来实例化一个Java 对象并填充字段值。如下所示：
```java
request.bodyHandler(buff -> {
  JsonObject jsonObject = buff.toJsonObject();
  User javaObject = jsonObject.mapTo(User.class);
});
```
在最简单的情况下，如果 Java 类中所有的字段都是 public（或者有 public 的 getter/setter）时，并且有一个 public 的默认构造函数（或不定义构造函数），mapFrom 和 mapTo 都应该成功。

只要不存在对象的循环引用，嵌套的 Java 对象可以被序列化/反序列化为嵌套的JSON对象。

#### Encoding a JSON object to a String 将 JSON 对象编码成字符串
您可使用 encode 方法将一个对象编码成字符串格式。

如要得到更优美、格式化的字符串，可以使用 encodePrettily 方法。

### JSON arrays JSON 数组
JsonArray 类用来描述 JSON数组。

一个JSON 数组是一个值的序列（值的类型可以是 string、number、boolean 等）。

JSON 数组同样可以包含 null 值。

#### Creating JSON arrays 创建 JSON 数组
可以使用默认构造函数创建空的JSON数组。

您可以从JSON格式的字符串创建一个JSON数组：
```java
String jsonString = "[\"foo\",\"bar\"]";
JsonArray array = new JsonArray(jsonString);
```

#### Adding entries into a JSON array 将数组项添加到JSON数组
您可以使用 add 方法添加数组项到JSON数组中：

#### Getting values from a JSON array
您可使用 getXXX 方法从JSON 数组中获取值。例如：
```java
String val = array.getString(0);
Integer intVal = array.getInteger(1);
Boolean boolVal = array.getBoolean(2);
```

#### Encoding a JSON array to a String
您可使用 encode 将一个 JsonArray 编码成字符串格式。

#### Creating arbitrary JSON
这里一直假设您在使用有效的表示形式
如果你不确定，你应该用Json.decodeValue
```java
Object object = Json.decodeValue(arbitraryJson);
if (object instanceof JsonObject) {
  // That's a valid json object
} else if (object instanceof JsonArray) {
  // That's a valid json array
} else if (object instanceof String) {
  // That's valid string
} else {
  // etc...
}
```

## Json Pointers
你可以用pointers来查询或写入。可以用字符串、URI或者人工添加地址来构造JsonPointer
```java
JsonPointer pointer1 = JsonPointer.from("/hello/world");
// Build a pointer manually
JsonPointer pointer2 = JsonPointer.create()
  .append("hello")
  .append("world");
```
pointer初始化之后，用`queryJson`来查询JSON值，用`writerJson`更新
```java
Object result1 = objectPointer.queryJson(jsonObject);
// Query a JsonArray
Object result2 = arrayPointer.queryJson(jsonArray);
// Write starting from a JsonObject
objectPointer.writeJson(jsonObject, "new element");
// Write starting from a JsonObject
arrayPointer.writeJson(jsonArray, "new element");
```
提供JsonPointerIterator实现后你可以对任意对象model使用Vert.x Json Pointer

## Buffers
在 Vert.x 内部，大部分数据被重新组织成 Buffer 格式。

一个 Buffer 是可以读取或写入的0个或多个字节序列，并且根据需要可以自动扩容、将任意字节写入 Buffer。您也可以将 Buffer 想象成字节数组（类似于 JDK 中的 ByteBuffer）。

### Creating buffers
可以使用静态方法 Buffer.buffer 来创建 Buffer。

Buffer 可以从字符串或字节数组初始化，或者直接创建空的 Buffer。

这儿有一些创建 Buffer 的例子。

创建一个空的 Buffer：
```java
Buffer buff = Buffer.buffer();
```
从字符串创建一个 Buffer，这个 Buffer 中的字符会以 UTF-8 格式编码：
```java
Buffer buff = Buffer.buffer("some string");
```
从字符串创建一个 Buffer，这个字符串可以用指定的编码方式编码，例如：
```java
Buffer buff = Buffer.buffer("some string", "UTF-16");
```
从字节数组 byte[] 创建 Buffer：
```java
byte[] bytes = new byte[] {1, 3, 5};
Buffer buff = Buffer.buffer(bytes);
```
创建一个指定初始大小的 Buffer。若您知道您的 Buffer 会写入一定量的数据，您可以创建 Buffer 并指定它的大小。这使得这个 Buffer 初始化时分配了更多的内存，比数据写入时重新调整大小的效率更高。注意以这种方式创建的 Buffer 是**空的**。它不会创建一个填满了 0 的Buffer。代码如下：
```java
Buffer buff = Buffer.buffer(10000);
```

### Writing to a Buffer

#### Appending to a Buffer

#### Random access buffer writes

### Reading from a Buffer

### Working with unsigned numbers

### Buffer length

### Copying buffers

### Slicing buffers

### Buffer re-use

## Writing TCP servers and clients

### Creating a TCP server

### Configuring a TCP server

### Start the Server Listening

### Listening on a random port

### Getting notified of incoming connections

### Reading data from the socket

### Writing data to a socket

### Closed handler

### Handling exceptions

### Event bus write handler

### Local and remote addresses

### Sending files or resources from the classpath

### Streaming sockets

### Upgrading connections to SSL/TLS

### Closing a TCP Server

### Automatic clean-up in verticles

### Scaling - sharing TCP servers

---

### Creating a TCP client

### Configuring a TCP client
Making connections
Configuring connection attempts
Logging network activity
Configuring servers and clients to work with SSL/TLS

#### Enabling SSL/TLS on the server

#### Specifying key/certificate for the server

#### Specifying trust for the server

#### Enabling SSL/TLS on the client

#### Client trust configuration

#### Specifying key/certificate for the client

#### Self-signed certificates for testing and development purposes

#### Revoking certificate authorities

#### Configuring the Cipher suite

#### Configuring TLS protocol versions

#### SSL engine

#### Server Name Indication (SNI)

#### Application-Layer Protocol Negotiation (ALPN)

##### OpenSSL ALPN support

##### Jetty-ALPN support

## Using a proxy for client connections