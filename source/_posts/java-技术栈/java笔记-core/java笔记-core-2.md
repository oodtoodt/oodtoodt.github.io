---
title: java笔记-core-2
date: 2020-04-07 09:54:19
tags:
- java
- company
- java-core

categories:
- java
- book_notes

---

接口、lambda、代理、异常
从善如流，没有看内部类（手动扇子）
<!--more-->

---

## 接口
在 Java 程序设计语言中， 接口不是类，而是对类的一组需求描述，这些类要遵从接口描述的统一格式进行定义。
所有的类都是默认实现了Object接口。
    
    有句话是这么说的：Object变量只能用于作为各种值的通用持有者。。。
    经过了一套实验，我发现数据是不会丢的，方法却是不会有的。
    和我的记忆一样，但我还真就找不到那句话了。
    所以强转不会丢数据(精度除外)

比如
```java
public interface Comparable
{
    int compareTo(Object other);
}
```
这就是说， 任何实现 Comparable 接口的类都需要包含 compareTo 方法，并且这个方法的参数必须是一个 Object 对象， 返回一个整型数值。
```java
public interface Comparable<T>
{
    int compareTo(T other);
}
```
从这开始java就和c++偏离的很远了，比如你可以看到java会大肆使用各种静态方法(不过c++中真的不会这样做吗？)但在现在的我看来这确实是很不一样的一点。
```java
java.lang.Comparable<T>
    int compareTo(T other)
java.util.Arrays
    static void sort(Object[] a)
java.lang.Integer
    static int compare(int x,int y)
```
接口不是类，尤其不能使用new运算符实例化一个接口。
但是却能声明接口的变量
接口变量必须引用实现了接口的类对象。
instance也可以检查对象是否实现了某个接口
接口也可以被扩展（继承）
注意，接口中的域被自动设为*public static final*

这个时候你会觉得interface特别像抽象类，但其实更抽象:比如不能有实现，不能构造，一个子类只能继承一个抽象类却能有多个接口，逗号隔开即可。

成了，这里还讲了为什么既然有了抽象类还要有接口：主要就是扩展只能一个。这里的主要问题是c++支持多重继承，而Java不支持。多继承会让语言本身变得非常复杂，比如c++的虚基类、控制规则和横向指针类型转换等复杂特性。

----------------------
### 关于工厂
工厂函数看上去有点像函数，实质上他们是类，当你调用它们时，实际上是生成了该类型的一个实例，就像工厂生产货物一样.

工厂方法
定义：定义一个创建对象的接口，但让实现这个接口的类决定实例化哪个类，工厂方法让类的实例化推迟到子类中进行。

-----------------------
### 默认方法
接口方法可以提供默认实现。
简单说，默认方法就是接口可以有实现方法，而且不需要实现类去实现其方法。
我们只需在方法名前面加个 default 关键字即可实现默认方法。

```java
public class t2 {
   public static void main(String args[]){
      Vehicle vehicle = new Car();
      vehicle.print();
   }
}
 
interface Vehicle {
   default void print(){
      System.out.println("我是一辆车!");
   }
    
   static void blowHorn(){
      System.out.println("按喇叭!!!");
   }
}
 
interface FourWheeler {
   default void print(){
      System.out.println("我是一辆四轮车!");
   }
}
 
class Car implements Vehicle, FourWheeler {
   public void print(){
      Vehicle.super.print();
      FourWheeler.super.print();
      Vehicle.blowHorn();
      System.out.println("我是一辆汽车!");
   }
}
```
这个例子非常有趣。

默认方法是重要用法是「接口演化」。指在接口中新增方法时保证源代码兼容（可以让类在不添加方法的情况下调用该新方法而不会出错。

解决默认方法冲突在java里很简单：首先超类优先，其次接口冲突。如果两个接口冲突就必须覆盖这个方法来解决冲突，（两个接口没有为共享方法提供默认实现，则无冲突）实现上可以选择一个。
```java
class Student implements Person, Named
{
    public String getName() { return Person.super.getName(); }
}
```
注意实现类不必实现所有的接口，那么它此时就是抽象类。

如果超类和接口继承了相同的方法，只会考虑超类方法，接口的所有默认方法都会被忽略。

--------------------
### 关于拷贝
clone 是Object的一个protected方法。这意味着你只能通过对象来clone对象。注意，clone是浅拷贝。
对于需要深拷贝的，我们的类必须实现Cloneable接口或者重新定义clone方法。
Cloneable接口是Java提供的一组标记接口之一。标记接口不包含任何方法，唯一作用就是允许在类型查询中使用instanceof。
建议自己的程序中不要使用标记接口

即如果你在一个对象上调用clone，但这个对象的类并没有实现Cloneable接口，Object类的clone方法就会抛出一个异常。

数组有public的clone。
有些人认为应该完全避免使用clone。毕竟，标准库只有5%的类实现了clone

--------------------
## lambda
lambda
```
java: () -> {}
c++:[]()mutable()constexpr exception attribute -> ret{};[](){};[]{};
```
java中，如果只有一个参数且类型可推导，则可以省略小括号
```java
ActionListener listener = event ->
    System.out.println("The time is " + new Date()");
    //Instead of (event) -> ... or (ActionEvent event) -> ...
```

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```
lambda在某些分支返回一个值，在另一些分支不返回值，是不合法的。
试了试，c++也不合法。而且这个例子也和c++差不多
```Java
Arrays.sort(planets,(first,second) -> first.length() - second.length());
Timer t = new Timer(1000,event -> System.out.println("the time is " + new Date()));
```


接口可以声明非抽象方法。同时，如果接口重新声明Object类的方法，如toString或clone，这些声明可能让方法不再是抽象的。

对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个lambda表达式（），这种接口称为函数式接口。
lambda可以转换为接口，这一点让lambda表达式很有吸引力
```java
Timer t = new Timer(1000, event ->
    {
        System.out.println("At the tone, the time is " + new Date());
        Toolkit.getDefaultToolkit().beep();
    }
)
```
java.util.function包中定义了很多通用的函数式接口，但是很多都没有什么帮助（。
这里提了一嘴Predicate，也不说干啥用的。

### 方法引用
```java
Timer t = new Timer(1000,event -> System.out.println(event));
//可以直接
Timer t = new Timer(1000,System.out::println);

System.out::println
//等价于
x -> System.out.println(x);
```
::操作符分隔方法名与对象或类名，有3种情况
object::instanceMethod
Class::staticMethod
Class::instanceMethod
前面2种，方法引用等价于提供方法参数的lambda表达式如上，对于第三种情况，第1个参数会成为方法的目标，`String::compareToTgnoreCase`等同于`(x,y) -> x.compareToIgnoreCase(y)`

如果有多个同名的重载方法，编译器就会尝试从上下文中找出那个方法。
```java
this::equals 
x -> this.equals(x);
```
同理super也是可以的。

-------------------
构造器也可以引用，方法名就是new。
可以用数组类型建立构造器引用，比如`int[]::new`等价于`x->new int[x]`
数组构造器对于克服 不能构造泛型类型T的数组 很有用。
假设我们需要一个Person对象数组
```java
Object[] people = stream.toArray();//stream接口有一个toArray方法可以返回Object数组
//但是用户希望得到一个Person引用数组，于是可以
Person[] people = stream.toArray(Person[]::new);
```
toArray方法调用这个构造器来得到一个正确类型的数组，然后填充这个数组并返回。

---------
#### lambda作用域
下面讨论lambda作用域的问题。
java的lambda是可以捕捉外围作用域中变量的值。是的，java也有闭包。同时，java的引用值不可以改变，内外改变都不合法。规则就是lambda中捕获的变量必须实际上是最终变量。
在lambda表达式中似乎用this关键字时，是指创建这个lambda表达式的方法的this参数

Runnable在java中很常用


------------
## 内部类
下面讲内部类
嵌套是一种类之间的关系，而不是对象之间的关系。嵌套类有两个好处，命名控制和访问控制。在java中前者并不重要，因为这是包管理的内容，后者确实只能靠内部类。
然而内部类还有另一个功能，使得它比c++的嵌套类更加丰富，用途更加广泛。内部类的对象有一个隐式引用，它引用了实例化该内部对象的外围类对象。通过这个指针，可以访问外围类对象的全部状态。
简单地说，static内部类（没有这种指针）与c++的嵌套类很类似。

内部类既可以访问自身的数据域，也可以访问创建它的外围类对象的数据域
使用外围类引用的正规语法是*OuterClass*.this.xxx方法
内部类中声明的所有静态域都必须是final。内部类不能有static方法。

内部类的语法非常复杂，甚至还有匿名内部类。
内部类是一种编译器现象，与虚拟机无关。

--------------
### 代理
newProxyInstance(类加载器（null就是默认），一个class对象数组，一个调用处理器)
代理类一定是public和final。
对于InvocationHandler接口的类对象（这个接口只有一个方法：
```java
Object invoke(Object proxy, Method method, Object[] args)
```
无论何时调用代理对象的方法，调用处理器的invoke方法都会被调用，并向其传递Method对象和原始的调用参数。
注意到下面的代码中，所有的代理操作其实都是在二分那一行代码里出现的。即操作的时候调用了invoke方法，并且用invoke方法输出了当前执行到这个代理上的操作的名字，实在是挺nb的。

```java

import java.lang.reflect.*;
import java.util.*;

public class t4
{
    public static void main(String[] args)
    {
        Object[] elements = new Object[1000];
        //fill elements with proxies for the integers 1 ... 1000
        for(int i = 0; i < elements.length; i ++)
        {
            Integer value = i + 1;
            InvocationHandler handler = new TraceHandler(value);
            Object proxy = Proxy.newProxyInstance(null, new Class[] { Comparable.class }, handler);
            elements[i] = proxy;
        }
        //construct a random integer
        Integer key = new Random().nextInt(elements.length) + 1;
        //search for the key
        int result = Arrays.binarySearch(elements, key);
        //print match if found
        if(result >= 0) System.out.println(elements[result]);
    }
}
class TraceHandler implements InvocationHandler
{
    private Object target;
    public TraceHandler(Object t)
    {
        target = t;
    }
    public Object invoke(Object proxy, Method m,Object[] args) throws Throwable
    {
        //print implicit argument
        System.out.print(target);
        //print method name
        System.out.print("." + m.getName() + "(");
        //print explicit arguments
        if(args != null)
        {
            for(int i = 0; i < args.length; i++)
            {
                System.out.print(args[i]);
                if(i < args.length - 1) System.out.print(",");
            }
        }
        System.out.println(")");
        //invoke actual method
        return m.invoke(target,args);
    }
}
/*
500.compareTo(386)
250.compareTo(386)
375.compareTo(386)
437.compareTo(386)
406.compareTo(386)
390.compareTo(386)
382.compareTo(386)
386.compareTo(386)
386.toString()
386
*/
```
#### 代理的意义
>我们为什么要引入java的代理，除了当前类能够提供的功能外，我们还需要补充一些其他功能。
最容易想到的情况就是权限过滤，我有一个类做某项业务，但是由于安全原因只有某些用户才可以调用这个类，此时我们就可以做一个该类的代理类，要求所有请求必须通过该代理类，由该代理类做权限判断，如果安全则调用实际类的业务开始处理。
可能有人说为什么我要多加个代理类？我只需要在原来类的方法里面加上权限过滤不就完了吗？
在程序设计中有一个类的单一性原则问题，这个原则很简单，就是每个类的功能尽可能单一。为什么要单一，因为只有功能单一这个类被改动的可能性才会最小。
如果你将权限判断放在当前类里面，当前这个类就既要负责自己本身业务逻辑、又要负责权限判断，那么就有两个导致该类变化的原因，现在如果权限规则一旦变化，这个类就必需得改，显然这不是一个好的设计。


如果将控制器定义一个接口，然后飞船直接实现这个接口，那跟之前飞船直接继承自控制器没有本质的区别，之所以不这样做正是如楼主所说的，控制器是飞船的一部分，不能说飞船是一个控制器，因此这里用继承不合适，用接口的话语义上可能更合适一点，但是接口的名字应该改成Controllable(可控制的)更符合语义。说白了就是你可以实现接口，这是你设计上的自由。但是这里可能用组合的方式更好。即有一个类SpaceShip（飞船），它包含一个成员SpaceShipController（控制器），然后飞船能做的控制上的事情就都可以通过这个控制器来做。
这里说的代理可以实现部分接口是根据具体场景来的，比如说现在你有一个控制器SpaceShipController，这个控制器提供了很多的功能，但实际你的SpaceShip可能并不需要这么多功能，如果你用SpaceShipController作为你飞船的成员变量的话，你的飞船里面就很有可能会去用到那些本来不需要的功能，由于你在SpaceShip里面确实能通过SpaceShipController去访问到那些功能，这可能不是一个好的设计原则，你实际上并不需要对飞船暴露那么多功能，这里你就可以使用一个代理SpaceShipControllerDelegation,用来代理那个SpaceShipController,然后在代理中只提供你需要的一些方法，最后在SpaceShip中不要用SpaceShipController作为成员变量，而是直接用那个代理作为成员变量，这样的话你的SpaceShip里面就访问不到SpaceShipController中的一些方法了。之所以说代理类能有更多的控制能力是说你代理类中调用方法的时候目前是用
```java
public void down(int velocity) {
        controls.down(velocity);
}
```
你可以在该方法实现的前后加上一些逻辑，这样就提供了更多的控制能力。比如改成：
```java
public void down(int velocity) {
        if(velocity < 10) {
        controls.down(velocity);
        }
}
```
这样即代表velocity<10的时候才能执行down的操作，这就是所谓的代理可以增加更多的控制能力

这里说的代理应当是代理的...另一种分支的应用，看一下就好。总得说，就是为其他对象提供一种代理以控制对这个对象的访问。
即静态代理。上面的newProxyInstance则是动态代理。

--------------------------------------------------------------
## 异常

如果出现错误，程序应该
+ 返回到一种安全状态，并能让用户执行一些其他的命令；或者
+ 允许用户保存所有操作的结果，并以妥善的方式终止程序
有这些问题值得注意：
1.用户输入错误
2.设备错误
3.物理限制
4.代码错误

所有异常对象都是派生于Throwable类的一个实例。即都是从此继承而来，下一层立即分解为Error和Exception，Exception派生于RuntimeException，另一支包含其他异常（以I/O错误为代表）
如果出现「RuntimeException异常，那么就一定是你的问题」是一条相当有道理的规则。
+ 错误的类型转换
+ 数组访问越界
+ 访问null指针

以下则不是非受查（unchecked）异常，属于受查异常。
- 试图在文件尾部后面读取数据
- 试图打开一个不存在的文件
- 试图根据给定的字符串查找Class对象，而字符串表示的类并不存在

-------------------------
在遇到下述4种情况应该抛出异常：
1. 调用一个抛出受查异常的方法，即方法本身已经throw了。
2. 运行过程中发生错误，利用throw抛出一个受查
3. 程序出现错误
4. 虚拟机和库出现的内部错误
前两种则必须告诉调用该方法的程序员可能会抛出异常。
即我们只关心受查异常，人家就应该被查，非受查请你更多的应该修正你的程序。
非受查要么不可控(Error)，要么应该避免。

>如果超类方法没有抛出异常，子类也不能。子类的受查异常不能比超类更通用。

java中的没有throws说明符的方法将不能抛出任何受查异常。
然而c++中的throw被抛弃了，换成了noexcept，即不抛出任何异常

-------------------
创建异常类是很正常的事情，派生于Exception（或者其子类）的类，应当有两个构造器，一个是默认的，一个带有详细描述信息

抛出很容易，因为抛出之后就不用了管了，捕获就会更复杂。
### try-catch
如果try语句块中的任何代码抛出了一个catch子句中说明的异常类，那么跳过try，执行catch。无异常跳过catch。
那么，什么时候该抛出，什么时候该捕获呢？
应该捕获那些知道如何处理的异常，将不知道怎么处理的继续传递。

可以抛出异常链，指的是捕获一个异常，然后用原始异常设置为新异常的原因，而且还能回到原始异常，这是一种包装技术，可以让用户抛出子系统的高级异常。
比如如果在一个方法中发生了一个受查异常，而不允许抛出它，包装技术就很有用。

抛出异常时会有关于方法之前一些资源的问题，这时候就需要使用finally子句。
不管是否有异常被捕获，finally子句中的代码都会被执行。
```java
InpuStream in = new FileInputStream(...);
try
{
    //1
    code..might throw exceptions
    //2
}
catch
{
    //3
    show error message
    //4
}
finally
{
    //5
    in.close();
}
//6
```
1. 代码无异常，1,2,5,6
2. 抛出一个在catch中捕获的异常。1)如果catch子句没有抛出异常，将执行1,3,4,5,6；2)如果catch抛出了异常，则为1,3,5
3. 代码抛出了异常，却不是catch捕获的，这时是1,5
try可以无catch，只有finally

**强烈建议解耦合try/catch 和try/finally 语句块**，可以提高代码清晰度。
```java
InpuStream in = ...;
try
{
    try
    {
        code that might throw exceptions
    }
    finally
    {
        in.close();
    }
}
catch(IOException e)
{
    show error message
}
```
内部try就是确保关闭输入流，外部try就是报告错误。
当finally中包含了return语句时，会有意想不到的结果
```java
public static int f(int n)
{
    try
    {
        int r = n * n;
        return r;
    }
    finally
    {
        if(n == 2) return 0;
    }
}
```
如果调用f(2)，那么try语句块的结果是r = 4.然而在真正返回前还要执行finally，0覆盖了4

但是finally可能会在里面继续抛出异常，这个时候事情就麻烦了起来。
于是有了带资源的try。
```java
try(Scanner in = new Scanner(new FileInputStream("/usr/sharedict/words"),"UTF-8");
    PrintWriter out = new PrintWriter("out.txt"))
{
    while(in.hasNext())
        out.println(in.next().toUpperCase());
}
```

#### 分析堆栈轨迹元素
```java
Throwable t = new Throwable();
StackTraceElement[] frames = t.getStackTrace();
for(StackTraceElement f : frames)
    System.out.println(f);
```
### 关于异常，你应该知道...
1. 异常不能代替简单的测试，效率天差地别（也就100倍起步吧，我也不知道这个是常数级别的大还是复杂度上的大，应该是都大）
基本规则是，只在异常的时候使用异常机制。
2. 不要过分细化异常。
把每一条语句分装在独立的try语句块之中会使得代码量急剧膨胀。
3. 利用异常层次结构
不要只抛出RuntimeException异常，寻找更加适当的子类或者创建自己的异常类。
不要只捕获Thowable异常，否则，会使程序代码更难读、更难维护。
考虑受查和非受查异常的区别。
将一种异常转换为另一种更适合的异常时不要犹豫。
4. 不要压制异常
在Java中，往往强烈地倾向关闭异常。即在catch中什么也不做。
5. 苛刻比放任更好
6. 不要羞于传递异常（5、6可以归纳为早抛出，晚捕获。）

-----------------------
### 使用断言。
断言机制允许在测试期间向代码中插入一些检查语句。当代码发布时，这些插入的检测会被自动的移走。
（就是说可以随时启用关闭，只需要编译指令即可。注意，这不需要重新编译，而是类加载器的功能。
c语言的assert宏将断言中的条件转换成一个字符串，失败时这个字符串就会被打印出来，java中条件不会自动成为错误报告，必须用assert 条件:表达式;后面的表达式来产生消息字符串。

什么时候使用断言？
+ 断言失败是致命的，不可恢复的错误
+ 断言检查只用于开发和测阶段（「在靠近海岸时穿上救生衣，但在海中央时就把救生衣丢掉吧」）

断言是一种测试和调试阶段所使用的战术性工具; 而日志记录是一种在程序的整个生命周期都可
以使用的策略性工具

------------------------------
### 使用日志
日志就是为了解决在有问题的代码中插入一些System.out.println方法调用来观察程序运行的麻烦问题而生的。日志有这些好处：
+ 可以很容易的取消或不取消日志
+ 可以简单的禁止日志记录的输出
+ 可以被定向到不同处理器
+ 日志记录可以采用不同的方式格式化
+ 日志记录器和处理器可以对记录过滤
+ 可以使用多个日志记录器


不得不说终于有一个这种东西了。我其实一直...需要一个这种东西的，不知道能让我以前打acm省下多少时间啊哎，那就该早学java的。嘤嘤嘤。c++的日志不是标准库的，但是也有不少好用的，应该是区别于不同应用平台需求之类的。主要是这样一个思想。

未被任何变量引用的日志记录器可能会被垃圾回收。建议用一个静态变量存储日志记录器的一个引用。

记录日志的常见用途是记录那些不可预料的异常，典型的是
```java
//void throwing(String className, String methodName, Throwable t)
//void log(Level l, String message, Throwable t)
if(...)
{
    IOException exception = new IOException("...");
    logger.throwing("com.mycompany.mylib.Reader","read",exception);
    throw exception;
}
//还有
try
{...}
catch(IOException e)
{
    Logger.getLogger("com.mycompany.myapp").log(Level.WARNING, "Reading image", e);
}//里面这个com.mycompany.mylib就是个具有层次结构的名字而已。
```

剩下的具体细化的可以先查了吧。比如如何过滤，如果配置，如何格式化，我们总结一下日志的使用：
1. 为一个简单的应用程序，选择一个日志记录器，并把日志记录器命名为与主应用程序包一样的名字，例如，com.mycompany.myprog，可以调用这样的方法得到日志记录器
```java
private static final Logger looger = Logger.getLogger("com.mycompany.myprog");
```
2. 在应用程序中安装一个更适合的默认的日志配置。
```java
if(System.getProperty("java.util.logging.config.class") == null
    && System.getProperty("java.util.logging.config.file") == null)
{
    try
    {
        Logger.getLogger("").setLevel(Level.ALL);
        final int LOG_ROTATION_COUNT = 10;
        Handler handler = new FileHandler("%h/myapp.log", 0, LOG_ROTATION_COUNT);
        // Looger.getLogger("").addHandler(handler)
    }
    cathc(IOException e)
    {
        logger.log(Level.SEVERE, "Can't create log file handler", e);
    }
}
```

3. 牢记INFO、WARNING、SEVERE的消息将显示到控制台上，程序员想要的日志记录可以设定为FINE。

-------------------
### 稍微总结
调试的方法：
1. 自定义类应当覆盖toString方法，以提供有用的类信息。（指Logger.getGlobal().info或者System.out.println)
2. 每个类中单独放置一个单独的main方法，对每个类进行单元测试
3. JUnit
4. 日志代理 如果Random类的nextDouble方法出了问题，可以以匿名子类的实例形式创建一个代理对象。
```java
Random generator = new
    Random()
    {
        public double nextDouble()
        {
            double result = super.nextDouble();
            Logger.getGlobal().info("nextDouble: " + result);
            return result;
        }
    };
```
当调用nextDouble()方法时，就会产生一个日志信息。
5. 利用Throwable类提供的printStackTrace方法，获得异常对象中的堆栈情况
```java
try
{
    ...
}
catch(Throwable t)
{
    t.printStackTrace();
    throw t;
}
//也可以插入下面这句来获得堆栈轨迹
Thread.dumpStack();
```
6. 堆栈情况可以发送到文件中(`printStackTrace(PrintWriter s)`)
7. 将错误信息保存在文件中(`java MyProgram 2>errors.txt`)
8. 让非捕获异常的堆栈轨迹出现在System.err中并不理想。比较好的方法可以调用静态的Thread.setDefalutUncaughtExceptionHandler方法
```java
Thread.setDefaultUncaughtExceptionHandler(
    new Thread.UncaughtExceptionHandler()
    {
        public void uncaughtException(Thread t, Throwable e)
        {
            save information in logfile
        }
    });
```
9. 用-verbose标志启动虚拟器来观察类的加载过程
10. -Xlint选项可以告诉编译器对一些普遍容易出现的代码问题进行检查
11. 可以找出虚拟机的操作系统进程ID，运行jconsole程序
12. 用jmap实用工具获得堆的转储
13. -Xprof运行虚拟机

（-X选项是没有正式支持的选项。
