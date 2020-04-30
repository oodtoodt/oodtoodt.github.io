---
title: blueprint-todo-note
date: 2020-04-30 11:25:37
tags:
---

整蒙了，人傻了，困顿了
<!--more-->

## 关于避免回调地狱
回调地狱基本上是异步避免不了要面对的问题，最后就是怎么解决的问题。

比如我来上两段(三段)代码
```java
//--hell
public void initData() {
    client.getConnection(conn -> {
        if(conn.succeeded()) {
            final SQLConnection connection = conn.result();
            connection.execute(SQL_CREATE, res -> {
                if (!res.succeeded()) {
                    res.cause().printStackTrace();
                }
                
                connection.close();
            });
        }
    });
}
```
大概应该是这样的，因为我也是比着画瓢，实践的话太过麻烦了。我们发现最后这个`});}});}`应当可以称作最初的回调地狱了。
因为嵌套过多，也无法重用，所以其实没有看起来的那么香(但是我确实觉得这种逻辑非常好，符合人思考的方向).
我们来看看vertx.core中的Future可以如何做
```java
@Override
public Future<Boolean> initData() {
  Future<Boolean> result = Future.future();
  client.getConnection(connHandler(result, connection ->
    connection.execute(SQL_CREATE, create -> {
      if (create.succeeded()) {
        result.complete(true);
      } else {
        result.fail(create.cause());
      }
      connection.close();
    })));
  return result;
}
private Handler<AsyncResult<SQLConnection>> connHandler(Future future, Handler<SQLConnection> handler) {
  return conn -> {
    if (conn.succeeded()) {
      final SQLConnection connection = conn.result();
      handler.handle(connection);
    } else {
      future.fail(conn.cause());
    }
  };
}
```
很好！现在代码可以重用了，而且可读性也没有降低什么，回调地狱的情况也随之减轻。
这里的代码应当还有很大的优化空间，因为这里用的Future版本应该比较老旧，如果使用compose等方法，我觉得这里可以更优化。但是因为代码逻辑的问题，我也不太想改它...
但是我们来看看更吓人的...rxjava2
不喜欢回调，那么对异步流处理最好的可能就是rx
```java
@Override
public Completable initData() {
    return client.rxGetConnection()
        .flatMapCompletable(connection -> connection.rxExecute(SQL_CREATE).doOnTerminate(connection.close()));
}
```
emm，你也许会反驳说着是封装，这根本不是简化，那确实我只能反驳反驳说上面其实也不过是封装。但是这确实吊啊。

## 关于idea..
好多坑啊。
其实主要是maven导入的问题，总感觉idea会记住某些神奇的配置然后将错就错，哪怕你改了他也无动于衷，于是诸如「找不到主类」「找不到符号」此类的问题层出不穷，它们共同的特点就是甩锅，把矛头指向别的奇怪的地方。
具体来说，如果某个依赖自己依赖了其他的依赖，除非必要，尽量不要覆盖他


要先ctrl进去download source才能看到javadoc。

当一层里面套了两个已经Deprecated的回调的时候，第二层就不吃了，开始疯狂划线

## rxjava
我其实真不会了...说一些要注意的点吧

map的层数和嵌套的问题，注意谁是谁的参数
idea有时会即时显示返回值，这还真挺好的

