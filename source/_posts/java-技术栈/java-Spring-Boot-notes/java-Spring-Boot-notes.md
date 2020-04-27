---
title: java-Spring-Boot-notes
date: 2020-04-08 11:04:37
tags:
- java
- company

categories:
- java
- frame
- spring boot



---

我放眼望去，满眼都是Auto几个大字。
大抵就是用的时候查一下，还不会就Stack Overflow请。
<!--more-->

---

[安装Spring Boot]<-安装maven<-安装choco
准确的说只需要jdk和maven就行了
写一份奇怪的pom.xml(Project Object Model),maven开始疯跑，最后success
？应该有warning啊，warning呢。
输入了错误的指令，然后maven居然搞得跟真的一样装了一堆依赖...真吊啊
写了一个奇怪的java
>@RestController，@RequesetMapping是固定（刻板）注解，而且是mvn的，不是Spring Boot的（所以该学的还是要学的），前者便于人理解代码顺便告诉Spring是web(@Controller)，后者是提供了路径信息，告诉Spring所有带`("/")`的HTTP请求应当被map到home方法。
>第二个类级别的注释是@EnableAutoConfiguration,让Spring Boot在基于加载的jar依赖来「猜测」你想要如何配置Spring。因为spring-boot-starter-web添加了Tomcat和SpringMVC，所以自动配置会假定你要开发一个网络应用然后套一下Spring的预设定。不仅仅是Starters里面的会被加在设定集里，其他的也可以。
>main方法就是启动方法，和java一样。我们的java方法调用SpringApplication的run。SpringApplication会引导应用启动Spring,也就启动了自动配置的Tomcat网络服务器。我们要传一个Example.class作为run的方法来告诉SpringApplication这是主要Spring组件。args也会被传递来公开命令行参数
然后mvn spring-boot:run或者直接用ide的run去跑，就可以访问localhost:8080来看hello world了

接下来是关于配置...
忘了写笔记真是误了大事。

### 构造系统
（Maven）
默认继承defaults from Spring Boot

<properties>可以覆写需要的依赖

<dependencyMangement>是管理器，如果dependencies里的dependency自己没有声明version元素，那么maven就会到dependencyManagement里面去找有没有对该artifactId和groupId进行过版本声明，如果有，就继承它，没有报错。如果自己声明了那就无视这个管理器
scope=import可以使得依赖不再单一继承
但这样就不能覆写属性了，你需要在管理器里，spring-boot-dependecies entry之前添加你需要的属性作为dependency

<plugins>标签可以打包项目为一个可执行的jar

`
mvn spring-boot:run
`
`
mvn package
//完全自包含可执行的jar
`

### 结构化代码
如果一个class没有在一个package里声明，它就应当在default package里。default package通常不鼓励使用。

@SpringBootApplication通常放在主类中，它隐式定义某些项目的一个基本「search package」，比如如果你写JPA应用，你应当用它来搜@Entity。
如果你不用它，那@EnableAutoConfiguration and @ComponentScan annotations 也可以。

### 配置类
更推荐用Java-based的配置而非xml的sources。我们通常推荐你的主要source是a single @Configuration class。main方法是首选的
搜索`Enable*`批注解决你可能的代码遗留的XML配置问题
#### Import 更多地Configuration类
你不需要把所有Configuration放在一个类里，@Import可以用来import更多地Configuration类，有选择的，你可以选择@ComponentScan来自动选择所有的Spring组件

#### Import XML 配置
用@Configuration，然后用@ImportResource

### 自动配置
Spring Boot会自动根据你的依赖来配置。
你需要选择是否允许自动配置，使用@EnableAutoConfiguration 或者@SpringBootApplication 给你的@Configuration类（只用一个）

#### 逐渐更换自动配置
自动配置是非侵入的，你不需要为更改自动配置后代码会玩完而害怕。比如如果你想加入自己的DataSource bean，那么默认的嵌入式数据库就会退出。
如果你需要了解正在应用哪些配置和原因，可以用--debug开关启动你的应用。这样可以启动调试日志。
#### 禁用特定的自动配置
你可以使用@SpringBootApplication的exclude属性来禁用他们
@SpringBootApplication(exclude ={DataSourceAutoConfiguration.class})
如果该类不在类路径中，则可以用excludeName属性并指定完全限定的名称。如果你更喜欢@EnableAutoConfiguration，exclude和excludeName也可用。最后，你还可以用spring.autoconfigure.exclude属性来控制禁用列表
（你可以都用
（不建议直接使用这些自动配置类
### Spring Beans和Dependency Injection(依赖注入)
如果将应用程序类放在根package中，你可以@ComponentScan而无参数，你的所有应用程序组件就会自动的注册为Spring Bean。
@Autowired进行构造函数注入，如果有构造函数则也可省略
注入，使用构造函数注入会使得构造的被标记为final，随后无法更改

### 使用 @SpringBootApplication 批注
许多Spring Boot开发人员喜欢他们的应用程序使用自动配置，组件扫描，并能够在其“应用程序类”上定义额外的配置。 单个@SpringBootApplication批注可用于启用这三个功能，即：
+ @EnableAutoConfiguration：启用Spring Boot的自动配置机制
+ @ComponentScan：在应用程序所在的软件包上启用@Component扫描（请参阅最佳实践）
+ @Configuration：允许在上下文中注册额外的bean或导入其他配置类

### 跑你的程序
打包成jar并使用嵌入式HTTP服务器的最大优势是可以像运行其他应用一样运行你的程序。调试也会很容易。
用Maven的话
```lua
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
--启用了远程调试支持的话
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myapplication-0.0.1-SNAPSHOT.jar
--最简单的的话
$ mvn spring-boot:run
--MAVEN_OPT操作系统环境变量
$ export MAVEN_OPTS=-Xmx1024m
```
### 开发工具
spring-boot-devtools模块可以包含在任何项目中。
运行完全打包的程序时将自动禁用它。重新打包的存档里不包含devtool。

#### 属性默认值
开发过程中的缓存可能会适得其反，所以
spring-boot-devtools模块自动禁用缓存选项。
缓存选项通常由application.properties文件中的设置配置
Web开发中如果你希望配置日志记录可以打开spring.http.log-request-details

#### 自动重启
每当classpath上的文件更改时，使用spring-boot-devtools的应用都会重新启动。
分叉...
LiveReload...
DevTools上下文的「关闭挂钩」会在重新启动期间关闭。

如果发现重新启动对于您的应用程序而言不够快，或者遇到类加载问题，则可以考虑从ZeroTurnaround重新加载技术，例如JRebel。 这些方法通过在加载类时重写类来使它们更适合于重新加载。
##### 条件评估的变动会被log
##### 排除资源
Thymeleaf模板可以就地编辑。默认情况下，更改`/ META-INF / maven，/ META-INF / resources，/ resources，/ static，/ public`或`/ templates`中的资源不会触发重新启动，但会触发实时重新加载。如果要自定义这些排除项，则可以使用`spring.devtools.restart.exclude`属性。例如，要仅排除`/ static`和`/ public`，可以设置以下属性：
```lua
spring.devtools.restart.exclude =static/ **，public/ **
```
