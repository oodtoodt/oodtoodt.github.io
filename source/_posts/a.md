services\static-data 第一个和第二个一起按照文档要求改好版号上传 第三个屏蔽字库，暂时用不到，都是客户端发过来的。
注意gld规范

services\snqu_utils 大佬写的上传，跑一下就可以上传到

common\snqu\games 是Glog

coscmd 上传文件，用的腾讯的一套服务器

苏老师，我对游戏发布的整个流程还是有些模糊：
1. 我们生成的jar和zip然后上传就可以了吗？然后这个上传时用到的是coscmd
2. 是测试那边的人帮忙打包是吗？

3. 本地测试要怎么做呢？

4. 怎么用vertx指令启动服务器？
5. gradle-task-distribution-distzip会给每个子项目都打包 应该用哪一个呢

6. 平时需要关注群里发的问题吗，比如昨天的clientIndex问题

7. 像Executor.waitIf这样的方法是要去了解实现还是知道怎么用就可以

8. Bnet是什么平台吗...?

问题有些多，您可能比较忙，空闲的时候答下就好


7.15


苏俊:
首先，整个发布流程，是先到测试服、然后QA服，如果有iOS/Android提审的话，再到提审服，都测试通过后，再到生产服

苏俊:
目前测试服、QA服，没有做区分

苏俊:
就是有三套服务器，测试、提审、线上。相应地，还分海外版、国内版。目前只有海外版

苏俊:
相应地，会有海外的测试、提审、生产。国内的测试、提审、生产。

苏俊:
国内版本目前有测试、提审，还没有生产服

苏俊:
打包到测试服、提审服时，我们自己上传就可以了。用 services\snqu_utils\upload_jar.py脚本

苏俊:
更新到生产服务器的时候，只有运维有权限。我们把打包好的zip包，传到cos里，由运维来更新

苏俊:
2. 打包肯定是我们开发来打包。测试只是测一下功能是不是正常




--------------------------


苏俊:
3、本地测试的话。需要你在本地机器上，装好相应的开发环境。mysql/redis/rabbitmq ，可以装到docker里，方便些。然后改对应你本地的配置文件，把数据库之类，指向你自己的服务。

苏俊:
4、vertx启动服务器  

苏俊:
[图片]

苏俊:
Main class:  com.bgs.application.blade.game.Launcher
VM options: -Dfile.encoding=UTF-8 -Dlog4j2.configurationFile=log4j2-local.xml -Dvertx.logger-delegate-factory-class-name=io.vertx.core.logging.SLF4JLogDelegateFactory -Dapplication.logName=game
Program arguments: run com.bgs.application.blade.game.Server
Workin directory: E:\projects\Blade\Blade_20200331_dev\blades-backend-20190325\services
其他的service大同小异。改下main class的名字，VMoptins ,program arguments

苏俊:
对比下截图，能看出来

苏俊:
5、distzip是每个服务一个zip包，所有服务都需要

苏俊:
所有的zip都需要

苏俊:
6、目前看下就可以了，知道有这么个问题。对代码有点熟悉时，有时间可以对着代码熟悉下

oodt:
但是那样的话不是会重复很多依赖吗..

苏俊:
是的。开发、提审的部署相同，相同的jar，都在同一个目录下。生产的部署，是每个service一个独立的文件夹。

苏俊:
每个service,都是独立的java命令启动的

苏俊:
7. 知道怎么用就行了，感兴趣的话，了解下实现。我也没去深入了解过

苏俊:
8. Bnet 是我们这游戏的CP。 我们是代理

苏俊:
162.14.16.220/150.109.150.169 root UpQBCu7886xRiJUS

苏俊:
香港测试服的账号，你可以登录上去看下

oodt:
好的

苏俊:
/opt/blade目录下

苏俊:
lib 是所有公用jar的目录

苏俊:
bin是各个service的目录



今日心得：不要困在整个大函数中。我其实之前往里找waitfor的时候就有这种感觉了，里面非常复杂且黑科技。有好几层的嵌套，全部理解相当的困难。现在我想找到为什么我在本地跑的时候他就知道是local，收获的话是确实是个非常紧密的部件工程，比如会用到.properties，这个properties可能是在其他的文件夹下(services\library-common\src\main\resources\blade-local.properties)，而且会知道用的是local，这时候是（可能）gradle在起作用，它调用的services:game:Launcher.main()里面会有很多的log4j的log，你会发现它们其实来自各地，比如：
```
services\library-common\src\main\java\com\bgs\library\common\configuration\CommandLineConfigurator.java:
[2020-07-15T14:20:03+0800] [main] [WARN] [configuration.CommandLineConfigurator] No visibility was passed in, using default visibility.

services\library-common\src\main\java\com\bgs\library\common\configuration\ServerConfigurator.java：
[2020-07-15T14:20:03+0800] [main] [INFO] [configuration.ServerConfigurator] Selected environment is: local
```
就这样找到了local，后面就会告诉你用到了game-local.properties了，于是配置什么的就都加载好了。
我现在怀疑就是gradle了，在gradlew.bat里有这样一句:if "%OS%"=="Windows_NT" setlocal
这样我就明白了，java -jar后面的参数是gradle给的

----------------

我佛了，我根本看不进代码和逻辑，还是得一边笔记一边看，不然注意力太分散了。

短的代码基本还是很简单的。比如getAccountById/device/Ids一般就似乎判断一下会不会有error(validatenonnull)，然后就可以getAccount了，最后ExecutorFactory似乎一定要跟一个run()，有待观察。

---

小tip:
+ 数据存在ctx里，这个ctx全名OperationContext操作上下文，可以setData(把指定的数据类型的数据存进去，还可以getData取出来)
+ 封装了流式操作，用ExecutorFactory。(表示链接在一起的步骤链，将按注册顺序执行，如果需要，可以返回最后一步结果的Future，以便与vertx正常链接（compose/map 链接）)
+ ValidationUtils中的valitdate是用来确认是否实现了validatable接口，然后根据这个接口来判断新的错误(两层thorw吧应该就算)

---


authAnonymous，第一个中函数，create a session在服务器上或者创建一个新的账号，这个账号还会和B.net账号link。
首先都在Executorfatory上做
+ 先验证error各种null/empty的情况
+ 找到账号。若无则创建一个，创建用createAnonymousAccount
+ 检验该账号是否被ban
+ 若无B.net账号，创建一个(匿名anonymous的)
    + 检查一下创建的sessiondetail，然后加进ctx里，顺便给ctx里的account赋一下这个bnetA
+ 更新账号link
+ 更新gld版本
+ 生成session
+ //==sujun==//session存到redis中
    + 用sessionprovider.saveSession，需要把session转为SessionCacheInfo
+ run()


出现了不认识的authFirstParty

然后这个throws好恐怖啊
我就是想知道这个validate到底是确定什么jb的。

auth+GameCenter/GooglePlay/Gplus/Kakao/Nintendo
都用到了authFirstParty，长得都一样，用到函数的参数不太一样，后面提。

link+GameCenter/GooglePlay/Nintendo
用linkFirstParty，在自己信息的基础上，还要3个String bnet账号/密码/登陆token
作用是link a user with 证书

linkBnet不同，是用linkBnetMasterAccount，还要带gamesenter/googleplay/nintendo的参数

linkForce+GameCenter/GooglePlay/Nintendo
用linkForceFirstParty，同上

linkForceBnet同上，多两个异常判定

---

refershToken
ExecutorFactory起手。
userinfo gldversion判断空
token必须经过2/3的生命周期才可以refresh，容易倒推得：tokenExpirationSeconds是应当死亡的时间（token到期时间） - 1/3 * 已经存活的时间 < 当前时间，则报错too soon（说明还没到2/3存活时间）
accountProvider.requireAccount，扔给ctx（上面也有过，就是找到账号）
处理账号link问题
获取Bnet权力，避免一直refresh而获得不到的情况
更新ban状态
更新gldversion

refreshSession，注意
生成session如果gld是新的,避免token带有旧版本gld
//==sujun==
session存到redis

---

createAnonymousAccount
创建一个匿名Blades账号。会创建一个匿名B.net账号然后link到blades账号上


好，爷又不会了。就离谱，我才反应过来这里的大部分东西都是接口变量，那他到底用谁的实现呢。
上面根本找不到就离谱。
上面找到的是ctx，ctx就算资源的一种了，那再想知道是谁在调用根本就是天方夜谭，怎么会有这种设计
我再想想。
可能就是不想让人知道是谁在调用吧，很成功.jpg。用的人必须注意不能用错。
这里有一个奇异的设计，就是用list来存某一个数据


---


另记：听了个c++热更新讲座，以为是c++黑科技操作，没想到更底层，是汇编改机器码，先用指令找到所有函数名，根据需要修改的函数名找到函数内存地址，找到后用c++函数改变可读写性，然后改写代码，内部汇编做跳转，执行新的函数。（大致是这套流程，前面没什么问题，到了改写代码突然就跟前面失联了，他讲的时候是中间讲的跳转，后面又说直接改整个内存页，反正我是没怎么懂）

草我刚刚反应过来，汇编改代码还真不是随便说说，而是用memset和memcpy直接改，太强了。前后做个可读写修改，不需要修改整个内存页，需要内存页的可写性就行。

总的来说，每张ppt都会都认识，但是还真的组织不成整个一套流程，不得不说还是挺nb的。
我会了.jpg

---
docker

docker run -dit --name Myrabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq
docker run -dit --name Myrabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:managemen


docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql

docker run -itd --name redis-test -p 6379:6370 redis

docker exec -it redis-test /bin/bash
redis-cli

mysql -h localhost -u root -p

-p

mv /etc/apt/sources.list /etc/apt/sources.list.bak
echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >/etc/apt/sources.list
echo "deb http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list
echo "deb-src http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
apt-get update 
apt-get install -y vim

"registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://registry.aliyuncs.com",
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn"
  ],

  文件夹内:
  /tools 代码
  /tests 似乎是读取测试数据用的
  /service 不用说了吧
  /prometheus 服务监控系统
  /portal 用Angular CLI的不知道干啥的
  /ping-network 代码
  /on :Tool to switch between AWS profiles
  /matchmaking-simulation 估计是pvp的东西?
  /local-servers 一些docker脚本，我先用了，应该没什么影响，毕竟叫local
  /libs service-api-3.32.0?啥
  /infra 是代码，但是是什么代码呢..
  /grafana grafana 是一款采用 go 的时序数据展示工具
  /docs 使用的外部工具用文档
  /deploy 部署用的吧
  /configs 配置，很怪，可能是整体的
  /cloudformation-templates ?

行吧，这个还得我自己来。

Communications link failure


---

在启动AdminService服务时出现Caused by: com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure，是因为我mysql里没有数据吗（划掉，没填mysql账号密码
Caused by: com.bgs.library.common.exceptions.ServiceErrorException: Request to receive messages from the SQS failed. [httpErrorCode=500, serviceId=21, errorCode=3, debugInformation=[queueName=taskprocessing]] 
orchestrator 端口冲突（划掉，我开了两个？

---

Admin 有问题 [2020-07-17T13:52:22+0800] [main] [DEBUG] [dal.aurora.MigrationUtils] Validating schema: admin
[MDC={}]
[2020-07-17T13:52:23+0800] [main] [ERROR] [admin.Launcher] Unable to validate database schema
[MDC={}]
org.flywaydb.core.api.FlywayException: Validate failed: Schema `admin` doesn't exist yet

Arena正常 8088
Authentication正常 8081
campaign正常 8092
campaignmessage正常 8093
ims 正常 8086
inventory 正常 8082
matchmaking 正常 8084
orchestrator 正常 8095

presence 正常->稍有问题: 8090 ?already inuse bind？
rms 正常->稍有问题: 8091 already in use bind
尝试kill pid 再次执行依然有问题。
是先
[2020-07-17T13:52:57+0800] [vert.x-eventloop-thread-1] [INFO] [core.BaseServer] Deploying server to port: 8090[MDC={}]
[2020-07-17T13:52:57+0800] [vert.x-eventloop-thread-0] [INFO] [core.BaseServer] Deploying server to port: 8090[MDC={}]
然后出的
[2020-07-17T13:52:57+0800] [vert.x-acceptor-thread-0] [ERROR] [impl.HttpServerImpl] java.net.BindException: Address already in use: bind

taskprocessing: 很有问题：Caused by: com.bgs.library.common.exceptions.ServiceErrorException: Request to receive messages from the SQS failed. [httpErrorCode=500, serviceId=21, errorCode=3, debugInformation=[queueName=taskprocessing]]

analytics 正常 8087

game [2020-07-17T13:47:46+0800] [main] [INFO] [snqu.glog.sender.GLogTCPSenderImpl] Connecting to GLOG server ... [MDC={}]
[2020-07-17T13:47:46+0800] [main] [ERROR] [game.Launcher] Error launching the server[MDC={}] 
java.lang.NullPointerException: no null host accepted
解决

flyway -table=_schema_history -schemas=admin -user=root -password=123456 migrate

---

