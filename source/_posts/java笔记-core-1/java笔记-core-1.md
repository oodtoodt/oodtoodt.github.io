---
title: java笔记-core-1
date: 2020-04-01 11:04:10
tags:
---
java笔记（简单）
```java
public class Classname
{
    public static void main(String[] args)
    {
        program statements
    }
}
```
下划线和0b这种东西...有点像c++17，不过好像没有0b？翻了一下，对不起，都有，不是下划线是分号线，c++14
然后指数p
0.125 = 2^-3 -> 0x1.0p-3
unicode转义序列会在解析之前得到处理。想成是某种宏替换即可，注意注释也会生效。

这个UTF16给我看傻了。
找到了比较靠谱的https://blog.csdn.net/wusj3/article/details/88641084
具体说就是尽量不要用char

java不区分声明和定义（？其实我觉得c++后面也不分了，有统一的初始化要求，但是没有强制的区别....?）

const是Java保留但没有被使用。java必须用final定义常量。不过final在c++中是禁止派生

一般而言整数转浮点会丢精度，但是int转double却不会。不要强转bollean

`>>>可以用来用0填充高位，>>使用符号位填充。不存在<<<。即是说>>对于负数生成的结果可能会依赖于具体的实现（c++），java则不会。`

Java不使用逗号运算符。不过for里可以有。

String没有提供用于修改字符串的方法。这一点很像lua了。
substring什么的都很像。

一定不要用==判断字符串是否相等！它只能确定两个字符串是否放在同一个位置上。因为虚拟机可以想象为把字符串放在公共的存储池中——共享。但是在java里这就像c++的未定义行为一样的发生随机间歇性bug。

length返回代码单元数量，charAt(n)返回位置n的的代码单元，codePointAt(index)返回码点，condePoints方法会更好用一些。
codePoints()将字符串的码点作为一个流返回，调用toArray将它们放在一个数组中

StringBuilder();
读取：Scanner in = new Scanner(System.in);

printf中参数索引从1开始

java中不允许嵌套块中重定义变量。

没有重载，不能重新定义+和*，没有给程序员这个机会，所以只能提供方法。

java数组和c++数组在堆栈上不同，在堆上的数组指针一样。
`int[] a = new int[100];\\java`
不同于
`int a[100];//c++`
而等同于
`int* a = new int[100];//c++`
另外java中的[]预定义为检查数组边界而无有指针运算。即不能通过+1等方式得到下一个元素

java的main方法中，程序名不在args数组中，而指的是参数

java的多维（不规则）数组其实就是一维数组...的扩展，数组的数组。
```java
double[] temp = balances[i];
balances[i] = balances[i+1];
balances[i+1] = temp;
```

```java
double[][] balances = new double[10][6];//java
//等同于
double** balances = new double*[10];
for(int i = 0; i < 10; i++) balances[i] = new double[6];
```

-------------------------------
一个对象变量，并没有，实际，包含一个对象。隐含的意味就是，不存在默认构造(指声明的时候就帮你初始化的操作)，也可以不把声明和初始化绑定在一起。
局部变量不会自动地初始化为null
java的对象可以被看作c++的对象指针，即Date Birthday(java) = Date* birthday(c++)

-------------------------------
更改器方法和访问器方法
在c++中，带有const后缀的方法就是访问器方法，默认就是更改器方法。java里这两种方法在语法上没有明显的区别
c++程序员最容易犯的错误就是忘记了new操作符
隐式参数可以不用this表示，直接用

-------------------------------
```c++
class Employee
{
    private Date hireDay;
    public Date getHireDay()
    {
        return hireDay;//Bad
    }
}
Employee harry = ...;
Date d = harry.getHireDay();
double tenYearsInMilliSeconds = 10 * 365.25 * 24 * 60 * 60 * 1000；
d.setTime(d.getTime() - (long)tenYearsInMilliSeconds);
//give Harry ten years of added seniority
```
注意不要编写返回引用可变对象的访问器方法，这破坏了封装性。但是另一个方面，这里的返回值就直接是引用，这点值得注意。
如果需要返回一个这样的引用，应该使用它的clone（某种意义上，就是右值）

-----------------------------
方法可以访问所属类的私有特性，即使是不同的变量，只要对象一样就行的。
对于可变的类，使用final修饰符可能会对读者造成混乱，例如
```java
private final StringBuilder evaluations;
evaluations = new StringBuilder();
//final关键字只是表示evaluations变量中的对象不会再指示其他StringBuilder对象，但这个对象可以更改
public void giveGoldStar(){
    evaluations.append(...);
}
```
静态方法是没有this参数的方法，这也和c++相同
但是语法书写上略有不同。在c++中，使用::操作符访问自身作用域之外的静态域和静态方法，如Math::PI。其实等到第三次c++重用static关键字，含义与之前完全不同，解释为属于类且不属于类对象的变量和函数，就和java相同了。
静态方法有一种常见的用途，静态工厂方法构造对象。

-------------------------
每一个类可以有一个main方法。
Java总是采用按值调用，方法不能修改传递给它的任何参数变量的内容。但是，但是对象引用的状态是可以改变的。
* java对对象采用的是引用调用 这个说法是不对的。
比如交换两个参数，swap(a,b)，交换的却是两个拷贝。所以最好的理解就是创建了一个拷贝，这两个东西指向的对象是同一个对象，对象的状态改变，即使参数变量不再使用，原来的变量依然指向该改变过的对象。

------------------------
方法的签名包括方法名和参数类型，不包括返回类型

域会被自动初始化为默认值，域也称成员变量。

我的理解不对，系统会提供一个无参数构造器。那单纯就是关于声明的问题了。

c++和java一样可以直接初始化实例域（成员变量）

c++和java一样可以在一个构造器里调用另一个构造器了，不过可能需要一些using和初始化列表的配合，语法不太一样。java是this(...)。

-------------------
调用构造器的具体处理步骤：
1.所有数据域被初始化为默认值（0，false，null）
2.按照在类声明中出现的次序，依次执行所有域初始化语句和初始化块
3.构造器第一行调用第二个构造器，执行第二个构造器主体
4.执行构造器主体

-------------------
import导入就像using namespace的感觉
而且还能import static

-----------------------
所有继承都是公有的。
super.XX()
这里的super就跟using的感觉一模一样了吧。或者说跟`::`的那个感觉一样。
在c++中，初始化列表语法调用的是超类的构造函数，而不调用super
java中，不需要将方法声明为虚拟方法，动态绑定是默认的处理方式，如果不希望一个方法虚拟，则可以标记它为final
java不支持多继承
is-a规则的另一种表述法是置换原则，比如可以将子类对象赋给超类，这在c++中也是一样的。
注意下面的情况：
```java
Manager boss = new Manager(...);
Employee[] staff = new Employee[3];
staff[0] = boss;
boss.setBonus(50);//ok
staff[0].setBonus(50)//Error
```
这是因为编译器只会将staff系列认为是Employee对象，而boss是Manager对象，虽然他们引用的是同一个对象。
当然，is-a不能倒过来用。

注意：数组引用的转换是合法却非常危险的
```java
Manager[] managers = new Manager[10];
Employee[] staff = managers;
staff[0] = new Employee("...",...);
managers[0].setBonus(100);//Oops!
```
牢记在数组中存放类型兼容的引用，或者就跟创建类型相同是了。

-----------------
让我们看一下java是如何调用方法的，这个过程就像理解c++的名称查找一样。假设x.f(args)，x声明为类C的一个对象
1.查看对象的声明类型和方法名。列举所有C类和C的超类中public中名为f的方法（超类的私有不能访问）
2.查看调用方法时提供的参数类型，如果有完全匹配，那就他了。由于存在类型转换，所以可能很复杂。
3.如果是private方法、static方法、final方法或者构造器，那么编译器可以准确的知道是哪个方法，这种调用叫静态绑定。对应的是调用方法依赖于隐式参数（this）的实际类型并且运行时动态绑定
4.程序运行，动态绑定调用方法时，一定调用与x所引用对象的实际类型最合适的那个类的犯法。假设x实际是D，D是C的子类，就依次在D、D的超类....类推下去里面找f(String)
由于每次都要搜索，开销相当大，所以虚拟机给每个类开了个方法表，列出所有方法的签名和实际调用的方法。


这里的静态和动态和c++一样理解就行，就是编译期是否可以知道。
说白了其实就是有这点区别，别的就靠完全匹配，多个或者没有就报错。

--------------------
final和c++的在阻止子类覆盖方法、阻止定义子类上是一样的。总之都是：确保不会在子类中改变语义。
这样看c++11这个应该就是从java来的。

---------------------
进行类型转换的唯一原因是：在暂时忽视对象的实际类型之后， 使用对象的全部功能

少用转换和instanceof，多去重新设计你的超类。想想你怎么学的effective c++

----------------------------
抽象上和c++一样，抽象基类。区别是c++是自己辨认，java可以明确写明。
需要注意， 可以定义一个抽象类的对象变量， 但是它只能引用非抽象子类的对象。 例如，
```java
Person p = new Student("Vinee Vu" , "Economics") ;
```
对于protected，java中受保护的部分对于所有子类和同一包中所有其他类都可见。
对本包可见则是——不需要修饰符。

-------------------------------

在雇员和经理的例子中， 只要对应的域相等， 就认为两个对象相等。如果两个 Manager对象所对应的姓名、 薪水和雇佣日期均相等， 而奖金不相等， 就认为它们是不相同的， 因此，可以使用 getClass 检测。
但是， 假设使用雇员的 ID 作为相等的检测标准， 并且这个相等的概念适用于所有的子类， 就可以使用 instanceof 进行检测， 并应该将 Employee.equals 声明为 final。
同理，继承中的interface也可能有问题（指compareTo的调换律），分为子类比价含义不同的提前检测情况；以及相同的提供超类compareTo方法并把它声明为final


使用`==` 比较基本类型域，使用 equals 比较对象域

-------------------------------
强烈建议为自定义的每一个类增加 toString 方法。这样做不仅自己受益， 而且所有使用这个类的程序员也会从这个日志记录支持中受益匪浅
以及equals hashcode需要的话也可以。

-----------------------------------
```java
ArrayList<Employee> staff = new ArrayList<>();
//伪auto+伪vector
```
vector重载了`[]`，但是java没有重载，必须调用显式的方法。此外，c++的stl是值拷贝，java是引用。

一旦能确保不会造成严重的后果， 可以用@SuppressWamings("unchecked") 标注来标记这个变量能够接受类型转换， 如下所示：
```java
@SuppressWarnings ("unchecked") ArrayList<Employee> result =(ArrayList<Employee>) employeeDB.find(query); // yields another warning
```

----------------------
自动装箱(autoboxing)和拆箱对应于对象包装器(wrapper)，即会自动的翻译：
```java
//int n = list.get(i).intValue();
int n = list.get(i);
```
可变参数，c++11。不过c++里那叫一个难啊，毕竟模板放在那。

枚举类，和c++差不多。

-----------------------------------
在 Java 程序设计语言中， 接口不是类，而是对类的一组需求描述，这些类要遵从接口描述的统一格式进行定义。
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
工厂函数看上去有点像函数，实质上他们是类，当你调用它们时，实际上是生成了该类型的一个实例，就像工厂生产货物一样.

工厂方法
定义：定义一个创建对象的接口，但让实现这个接口的类决定实例化哪个类，工厂方法让类的实例化推迟到子类中进行。

-----------------------
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
clone 是Object的一个protected方法。这意味着你只能通过对象来clone对象。注意，clone是浅拷贝。
对于需要深拷贝的，我们的类必须实现Cloneable接口或者重新定义clone方法。
Cloneable接口是Java提供的一组标记接口之一。标记接口不包含任何方法，唯一作用就是允许在类型查询中使用instanceof。
建议自己的程序中不要使用标记接口

即如果你在一个对象上调用clone，但这个对象的类并没有实现Cloneable接口，Object类的clone方法就会抛出一个异常。

数组有public的clone。
有些人认为应该完全避免使用clone。毕竟，标准库只有5%的类实现了clone

--------------------
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

方法引用
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
下面讨论lambda作用域的问题。
java的lambda是可以捕捉外围作用域中变量的值。是的，java也有闭包。同时，java的引用值不可以改变，内外改变都不合法。规则就是lambda中捕获的变量必须实际上是最终变量。
在lambda表达式中似乎用this关键字时，是指创建这个lambda表达式的方法的this参数
Runnable在java中很常用

------------
下面讲内部类
嵌套是一种类之间的关系，而不是对象之间的关系。嵌套类有两个好处，命名控制和访问控制。在java中前者并不重要，因为这是包管理的内容，后者确实只能靠内部类。
然而内部类还有另一个功能，使得它比c++的嵌套类更加丰富，用途更加广泛。内部类的对象有一个隐式引用，它引用了实例化该内部对象的外围类对象。通过这个指针，可以访问外围类对象的全部状态。
简单地说，static内部类（没有这种指针）与c++的嵌套类很类似。

内部类既可以访问自身的数据域，也可以访问创建它的外围类对象的数据域
使用外围类引用的正规语法是*OuterClass*.this
内部类中声明的所有静态域都必须是final。内部类不能有static方法。

内部类的语法非常复杂，甚至还有匿名内部类。
内部类是一种编译器现象，与虚拟机无关。

--------------
代理
newProxyInstance(类加载器（null就是默认），一个class对象数组，一个调用处理器)
代理类一定是public和final。
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
```

--------------------------------------------------------------
异常

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

===

以下则不是非受查（unchecked）异常，属于受查异常。
+ 试图在文件尾部后面读取数据
+ 试图打开一个不存在的文件
+ 试图根据给定的字符串查找Class对象，而字符串表示的类并不存在

-------------------------
