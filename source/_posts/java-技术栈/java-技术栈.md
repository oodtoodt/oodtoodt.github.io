---
title: java-技术栈
date: 2020-04-07 15:03:50
tags:
---

别问我为啥需要这么多...
<!--more-->

---

## Spring Boot
如果说SSM是Spring IoC+SpringMVC+Mybatis的组合，是三个臭皮匠，那么Spring Boot就是诸葛亮。有了诸葛亮，出兵方案更多，出师更顺利，没有和任何MVC框架绑定，没有和任何持久层框架绑定，没有和任何其他业务领域的框架绑定。
Spring Boot提供的只是这些starters，这些Starter一来了对应的框架或技术，但不包含对应的技术或框架本身。
所以全家桶是不恰当的形容，因为全家桶是**包含**在里面的。
更恰当的比喻是，Spring MVC、Spring Data、Websocket这东西对应电脑硬件上的显卡声卡硬盘网卡，而Spring Boot提供的Staters对应这些硬件的驱动。只要你主板上有这些硬件，你就可以即插即用。
那Spring Boot和SSM相比，打算把开发者引导到什么方向上去呢?其中重要的一点就是：抛弃服务器模板，用REST API和前端配合，做到前后端分离的开发。
## SSM
### Spring
看了很多，大抵是说核心在于IoC+AOP,即控制反转和面向切面编程
#### Servlet
由服务器调用的，运行在服务器端的，一种类，可以处理浏览器带来的HTTP请求，并返回一个响应给浏览器，从而实现浏览器和服务器的交互
##### Tomcat
是一个Web服务器，可以很方便的**接受和返回**到请求（如果不用Tomcat，我们需要自己写Socket来接受和返回请求）。是Serlvet的一个容器
Tomcat即使提供让别人访问自己写的页面的一个程序，他就在浏览器
##### 
#### AOP

#### IoC
Ioc—Inversion of Control，即“控制反转”，不是什么技术，而是一种设计思想。在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。
是一个解耦合的过程，通过从其他地方定义对象（对象的具体注入），来保证开关原则（主业务对修改关闭，对扩展开放）核心机制建立在Java的反射上。

### Spring MVC
建立在IoC和Servlet之上。
C，控制器。M，模型。V，视图。就是一个交互的流程体系
### MyBatis
帮你和数据库打交道的框架，允许你像写java一样操作数据库，帮你把数据库的表翻译成类，字段翻译成类的字段，记录翻译成对象等等。
## MySQL
## Redis
Redis是一个开源的，基于内存的数据结构存储，可用作于数据库、缓存、消息中间件。
### 为什么用Redis
Redis实现的是分布式缓存，如果有多台实例(机器)的话，每个实例都共享一份缓存，缓存具有一致性。
专业做缓存，几十G也不在话下
### 为什么用缓存
这些一级缓存、二级缓存....归根结底这些本地缓存的目的就是为了：
不用每次读取的时候都要查一次数据库
提高了性能、提高了并发能力
### Redis的数据结构
"Redis is written in ANSI C"
常用的有string,list,hash,set,sortset
Redis的存储是以key-value的形式的，key一定是字符串。
值得注意的是，Redis并没有直接使用这些数据结构来实现key-vlaue，而是基于这些数据结构创建了一个对象系统
## RabbitMQ
## Spring Cache
## WebSocket

## WebJars
## Bootstrap
## Thymeleaf
## jQuery