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
静态方法不能向对象实施。

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

java还可以有初始化块，可以有多个块，注意如果加static的话，那么它就会在类加载的时候调用，并只初始化这一次。先调用父类的static{}再是子类。构造方法就是这样的（。

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

反射机制可以用来：
+ 在运行时分析类的能力
+ 在运行时查看对象
+ 实现通用的数组操作代码
+ 利用Method对象，这个对象很像c++中的函数指针

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
使用外围类引用的正规语法是*OuterClass*.this.xxx方法
内部类中声明的所有静态域都必须是final。内部类不能有static方法。

内部类的语法非常复杂，甚至还有匿名内部类。
内部类是一种编译器现象，与虚拟机无关。

--------------
代理
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
代理的意义
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
try-catch
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

分析堆栈轨迹元素
```java
Throwable t = new Throwable();
StackTraceElement[] frames = t.getStackTrace();
for(StackTraceElement f : frames)
    System.out.println(f);
```
关于异常，你应该知道...
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
使用断言。
断言机制允许在测试期间向代码中插入一些检查语句。当代码发布时，这些插入的检测会被自动的移走。
（就是说可以随时启用关闭，只需要编译指令即可。注意，这不需要重新编译，而是类加载器的功能。
c语言的assert宏将断言中的条件转换成一个字符串，失败时这个字符串就会被打印出来，java中条件不会自动成为错误报告，必须用assert 条件:表达式;后面的表达式来产生消息字符串。

什么时候使用断言？
+ 断言失败是致命的，不可恢复的错误
+ 断言检查只用于开发和测阶段（「在靠近海岸时穿上救生衣，但在海中央时就把救生衣丢掉吧」）

断言是一种测试和调试阶段所使用的战术性工具; 而日志记录是一种在程序的整个生命周期都可
以使用的策略性工具

------------------------------
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

剩下的具体细化的可以先查了吧。比如如何过滤，如果配置，如何格式化，我们总结一下：
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

-------------------------------------
泛型

```c++
g(f<a,b>(c))
```
如果c++将类型参数放在方法名的后面，有可能导致语法分析的歧义，比如这里可以理解成两个布尔值调用g或者是f的结果调用g。
那么java呢？java会放到前面，所以根本不这么写，而是
```java
g(<a,b>f(c));
```
但是大部分泛型方法的类型引用没有问题，所以编译器有足够的信息可以推断出调用的方法，所以一般这就省略了。

限定类型用&分割，类型变量用逗号分割

限定可以多借口，但是最多一个类。类限定必须是列表中的第一个

在虚拟机中，用参数类型和返回类型来确定方法签名，因此，编译器可以产生两个仅返回类型不同的方法字节码，虚拟机能正确处理这种情况。
说这个干什么呢，是因为有个神奇的东西叫桥类型。
泛型方法和多态冲突的情况会被编译器自动生成桥类型来纠正。
而且桥方法还可以在一个方法覆盖另一个方法时指定一个更严格的返回类型（因为这个例子短，所以放这个例子了，想不通泛型的话自己查或者多想一下）
```java
public class Employee implements Cloneable
{
    public Employee clone() throws CloneNotSupportedException{...}
}
```
实际上，Employee类有两个克隆方法
```java
Employee clone()//define above;
Object clone()//synthesized bridge method,overrides Object.clone
```
下面就是桥了，调用了新定义的方法。

+ 虚拟机中无泛型，只有普通的类和方法（他们被擦除了，只是最后会做强转而已）
+ 所有类型参数用限定类型体寒
+ 桥方法被合成来保持多态
+ 保持类型安全，必要时插入强制类型转换

    有句话是这么说的：Object变量只能用于作为各种值的通用持有者。。。
    经过了一套实验，我发现数据是不会丢的，方法却是不会有的。
    和我的记忆一样，但我还真就找不到那句话了。
    所以强转不会丢数据(精度除外)
这样的话很多东西都解释的通了，同时，类型擦除的特性导致出了各种限制:
+ 不能用基本类型实例化类型参数
很显然，基本类型没有Object这样的祖宗。
+ 运行时类型查询只适用于原始类型
很显然，每个对象总有一个特定的非泛型类型。
+ 不能创建参数化类型的数组
指的是
```java
Pair<String>[] table = new Pair<String>[10];//Error
```
主要是擦除之后table是`Pair[]`类型，可以把它转换为`Object[]`,数组会记住它的元素类型，如果试图存其他的元素（指String），就会出现异常。更绝的是如果你放泛型进去是会被擦除拯救，通过数组存储检查，但仍会在其他地方导致类型错误。当然，声明`Pair<String>[]`的变量是合法的，不能用`new Pair<String>[10]`去初始化它就是了
如果你一定要这种东西...使用`ArrayList:ArrayList<Pair<String>>`

+ Varargs警告
向可变参数的方法传递一个泛型类型...会得到一个警告（但是没有问题的，只要你写的是对的...）

+ 不能实例化类型变量
即
```java
new T(...),new T[...] or T.class这样表达式中的类型变量
public Pair() {first = new T(); second = new T();}//Error！
```
这意味着new Object()
最好的解决办法是让调用者提供一个构造器表达式
```java
Pair<String> p = Pair.makePair(String::new);
```
makePair方法接受一个`Supplier<T>`，这是一个函数式接口，表示一个无参数且返回类型为T的函数
```java
public static <T> Pair<T> makePair(Supplier<T> constr)
{
    return new Pair<>(constr.get(), constr().get());
}
```
+ 不能构造泛型数组
```java
public static <T extends Comparable> T[] minmax(T[] a) {T[] mm = new T[2]; ...};//ERROR!
```
类型擦除会让这个方法永远构造`Comparable[2]`数组
如果数组仅仅作为一个类的私有实例域，就可以先声明为`Object[]`,，然后在获取元素时进行类型转换
```java
public class ArrayList<E>
{
    private Object[] elements;
    ...
    @SuppressWarnings("unchecked") public E get(int n) {return (E) elements[n]; }
    public void set(int n,E e) { elements[n] = e; }//no cast needed
}
```
强制类型转换`E[]`是个假象，而类型擦除使其无法察觉。
由于minmax返回`T[]`数组，使这一技术无法施展。
最好让用户提供一个数组构造器表达式
```java
public static <T extends Comparable> T[] minmax(IntFunction<T[]> constr,T...a)
{
    T[] mm = constr.apply(2);
}
String[] ss = ArrayAlg.minmax(String[]::new,"Tom","Dick","Harry");
```
+ 泛型类的静态上下文中类型变量无效
不能在静态域或方法中引用类型变量（T）
+ 不能抛出或捕获泛型类的实例
甚至扩展Throwable都是不合法的。
不过异常规范里是可以使用类型变量的
```java
public static <T extends Throwable> void doWork(T t) throws T //ok
{
    try
    {

    }
    catch(Throwable realCause)
    {
        t.initCause(realCause);
        throw t;
    }
}
```
+ 消除对受查异常的检查
```java
//就用上面那个
Block.<RuntimeException> throwAs(t);
```
编译器就会认为t是一个非受查异常。这有什么意义呢？正常情况下，你必须捕获线程某方法中的所有受查异常，把他们包装到非受查异常中，因为该方法声明为不任何受查异常
通过泛型类、擦除和@SuppressWarnings注解，就能消除Java类型系统中的部分基本限制。
+ 注意擦除后的冲突
简单的冲突就
```java
public class Pair<T>
{
    public boolean equals(T value) { return first.equals(value) && second.equals(value);}
}
```
因为把string擦了之后就是object了，然后就是一模一样的equals冲突。重命名就完事了。
复杂的则牵扯到一个原则：需要强行限制一个类或类型变量不能同时成为两个接口类型的子类，而这两个接口是同一接口的不同参数化。
```java
class Employee implements Comparable<Employee> { ... }
class Manager extends Employee implements Comparable<Manager> { ... } //Error!
```
Manager会实现`Comparable<Employee> 和 Comparable<Manager>`，这是同一接口的不同参数化。

------------
泛型的继承
无论S和T有什么关系，通常`Pair<S>和Pair<T>`没有什么联系。
简单的例子，给Manager的Pair强转之后加一个Employee，这一对就很怪异了（当然是不可以的）
虽然数组可以这么做(指强转过去)，但是数组会在存储的时候抛出异常（指数组有特别的保护)
泛型可以扩展，扩展就可以转换，但是只能是`A<M>到B<M>`，而不能是`A<M>到A<E> or B<E>`。
可以将参数化类型转化为一个原始类型，之后会正常产生类型错误。

-----------------------------
通配符类型
固定的泛型类型使用起来并没有那么令人愉悦。于是java的设计者发明了一种巧妙的东西。
```java
Pair<? extends Employee>
```
表示任何泛型Pair类型，它的类型参数是Employee的子类，如`Pair<Manager>`，但不是`Pair<String>`
假设现在用Manager初始化一个pair，然后赋给这个东西,再setFirst,会有问题嘛?
你会发现出现了Compile-time Error.
于是你惊讶的发现setFirst不行,getFirst却可以,神奇的产生了分离.

通配符限定还可以值顶超类型限定,即? super Manager.这个通配符限制为Manager的所有超类型.注意带超类型限定的通配符不能使用返回值，可以为方法提供参数
一个例子：经理数组，想把奖金最低的和最高的经理放在一个Pair对象中，Pair的类型是什么？Employee是合理的，Object也是合理的。
```java
public static void minmaxBonus(Manager[] a,Pair<? super Manager> result)
{
    if(a.length == 0) return;
    Manager min = a[0];
    Manger max = a[0];
    for(int i = 1; i < a.length; i++)
    {
        if(min.getBonus() > a[i].getBonus()) min = a[i];
        if(max.getBonus() < a[i]Bonus()) max = a[i];
    }
    result.setFirst(min);
    result.setSecond(max);
}
```

```java
public static <T extends Comparable<T>> T min(T[] a)
//显然存在一种
public static <T extends Comparable<? super T>> T min(T[] a)
```
子类型限定的另一种常见用法是作为一个函数式接口的参数类型，比如Collection接口有一个方法，可以删除所有给定谓词条件的元素
```java
defalut boolean removeIf(Predicate<? super E> filter)
ArrayList<Employee> staff = ...;
Predicate<Object> oddHashCode = obj ->obj.hashCode() % 2 != 0;
staff.removeIf(oddHashCode);
```
可以看到，传入的是`Predicate<Object>`

无限定通配符，例如`Pair<?>`，它和Pair本质不同在于：*可以用任意Object对象调用原始Pair类的setObject方法*。这种类型对于许多简单的操作非常有用，比如测试一个pair是否包含一个null引用
```java
public static boolean hasNulls(Pair<?> p)
{
    return p.getFirst() == null || p.getSecond() == null;
}
//可以看成是下面的语法糖
public static <T> boolean hasNulls(Pair<T> p)
{
    ...
}
```
通配符捕获就是通过某种方式使得`?`的类型被确定的方法。比如如果写一个`public static void swap(Pair<?> p)`，不能`? t`，所以需要写一个`public static<T> void swapHelper(Pair<T> p)`来捕获这个通配符。这个例子的通配符不是必要的，但有时候，通配符捕获是不可避免的：
```java
public static void maxminBonus(Manager[] a, Pair<? super Manager> result)
{
    minmaxBonus(a,result);
    PairAlg.swap(result);//ok--swaphelper captures wildcard type
}
```
注意通配符捕获有很多的限制才会合法，编译器必须确信通配符表达的是单个、确定的类型。

------------------------------
循环的多种方式：
```java
Iterator<String> iter = c.iterator();
while(iter.hasNext()) {String element = iter.next();}
//对于所有实现了public interface Iterable<E>接口的对象
iterator.forEachRemaining(element -> do something with element);
for(String element:t){}
```

java集合类库中的迭代器和其他类库中的迭代器在概念上有着重要的区别。c++的stl中，迭代器是根据数组索引建模的，可以直接查看指定位置上的元素，不需要查找元素就可以将迭代器向前移动一个位置。但是Java迭代器不是。它的查找和位置变更时相连的。**可以将java迭代器认为是位于两个元素之间**。
remove可以删除上次next方法返回的元素。（没有之前next的remove是不合法的）

集合类的基本借口是Collection接口，里面其实有很多的方法。
实际上有两种有序集合，数组支持的有序集合可以快速随机访问，适合用list并提供一个整数索引来访问。链表尽管也有序，但是随机访问很慢，应使用迭代器遍历。（可以用instanceof RandomAccess是否true测试是否支持高校访问）
java和c的list含义完全不一样有点难过吧
java              c++
ArrayList -- vector(是真的，是数组实现的)
LinkedList -- 双向链表
ArrayDeque -- 双向队列(Deque)
HashSet -- hash_set
TreeSet -- set
TreeMap -- map(都是红黑树)
HashMap -- hash_map
EnumSet 
LinkedHashMap -- Hashmap+双向链表，可以维护插入顺序
WeakHashMap -- key引用对象可能被回收，回收后即删除该键值对
IdentityHashMap -- 严格相等(==而非equals)
PriorityQueue -- 、

为了避免发生并发修改的异常，请遵循简单规则：可以给句需要给容器附加许多的迭代器，但是只读。单独附加一个既读又写的迭代器。
集合可以跟踪改写操作次数，每个迭代器可以维护一个独立的计数值，不一致抛出异常。（这个跟踪不会算set方法。）
使用链表的唯一理由是尽可能减少列表中插入删除的代价（废话
不需要同步时用ArrayList，Vector是同步的，
```java
Set<K> keySet();
Collection<V> values()
Set<Map.Entry<K,V>> entrySet();
```
注意Entry是个接口，不能像c++那样实例化
所以说还是Pair好用？

-------------------------------------------------------
视图与包装器
视图是这样一种集合：keySet方法返回一个实现Set接口的类对象，这个类的方法对原映射进行操作。
将集合类对象中的数据重新映射到一个数据集合中，但是这个集合不是以物理上存在的对象实体，而是再映射，物理地址不变，访问数据的接口变了。
Arrays类的静态方法asList将返回一个包装了普通java数组的List包装器，这样就可以将数组直接传递给列表或集合参数的方法了。
注意返回的对象不是ArrayList。而是一个带有访问底层数组get和set方法的视图对象。
通常，视图有一些局限性，即只可以读、无法改变大小、支持删除而不支持插入。
返回空类型有的时候很有用。`Set<String> deepThought = Collections.emptySet()`
不可修改视图、同步视图、受查视图。

----------------------------
反射
+ 在运行时分析类的能力
+ 在运行时查看对象，例如编写一个toString方法供所有类使用
+ 实现通用的数组操作代码
+ 利用Method对象，这个对象很像C++中的函数指针

有一个专门的Java类访问java为所有对象维护的信息，他就是Class
最常用的方法就是getName，会返回累的名字
静态方法forName获得类名对应的Class对象，注意提供异常的情况
如果T是java类型，T.class代表匹配的类对象
一个Class对象实际上表示的是一个类型，而这个类型未必是一种类，T.class的类型其实就是`Class<T>`
`class.getDeclaredConstructor().newInstance()`可以用来动态的创建一个类的实例，调用默认的构造器

Field,Method,Constructor用于描述类的域、方法和构造器。其中，Modifier类可以分析getModifier方法返回的值以描述public和static这样的修饰符情况
注意，get和getDeclare前缀是不同的，比如geteDeclareMethods返回Class对象表示的类和接口的所有已声明的方法数组，不包括父类继承和接口实现的方法。getMethods返回当前Class对象表示的类或接口的所有公有成员方法对象数组，包括已声明的和从父类继承或实现接口的方法。

如果一个java没有受到安全管理器的控制，就可以覆盖访问控制，setAccessible可以做到。
我们在这里提供一个通用的toString方法，可供任意类使用的通用toString方法。

```java
//import...
import java.lang.reflect.AccessibleObject;
import java.lang.reflect.Field;
import java.lang.reflect.Array;
import java.lang.reflect.Modifier;
import java.util.ArrayList;

public class t8
{
    public static void main(String[] args)
    {
        ArrayList<Integer> squares = new ArrayList<>();
        for(int i = 1; i <= 5; i++){squares.add(i * i);}
        System.out.println(new ObjectAnalyzer().toString(squares));
    }
    static class ObjectAnalyzer
    {
        private ArrayList<Object> visited = new ArrayList<>();
        /**
         * Converts an object to a string representation that lists all fields
         * @param obj an object
         * @return a string with the object's class name and all field names and
         * values
         */
        public String toString(Object obj)
        {
            if(obj == null) return "null";
            if(visited.contains(obj)) return "...";
            visited.add(obj);
            Class cl = obj.getClass();
            if(cl == String.class) return (String) obj;
            if(cl.isArray())
            {
                String r = cl.getComponentType() + "[]{";
                for(int i = 0; i < Array.getLength(obj); i++)
                {
                    if (i > 0) r += ",\n";
                    Object val = Array.get(obj,i);
                    if(cl.getComponentType().isPrimitive()) r += val;
                    else r += toString(val);
                }
                return r + "}";
            }

            String r = cl.getName();
            //
            do
            {
                r += "[";
                r += cl.getName() + " . ";
                Field[] fields = cl.getDeclaredFields();
                AccessibleObject.setAccessible(fields, true);
                //get the names and values
                for(Field f : fields)
                {
                    if(!Modifier.isStatic(f.getModifiers()))
                    {
                        if(!r.endsWith("[")) r += ",";
                        r += f.getName() + "=";
                        try
                        {
                            Class t = f.getType();
                            Object val = f.get(obj);
                            if(t.isPrimitive()) r += val;
                            else r += toString(val);
                        }
                        catch(Exception e)
                        {
                            e.printStackTrace();
                        }
                    }
                }
                r += "]";
                cl = cl.getSuperclass();
            }
            while(cl != null);
            return r;
        }
    }   
}
/**java.uti1.ArrayList[elementData=class java.1ang.Object[] {java.1ang.Integer[value=1][][],
 * java.1ang.Integer[value=4][][],java.1ang.Integer[value=9][][],java.1ang.Integer[value=16][][],
 * java.1ang.Integer[value=25][][],null,null,null,null.null},size=5][modCount=5][][]
 * /
```
不要对那些`[][]`感到惊奇。Integer的超类是Number，Number的超类是Object。最后那里则是`[AbstractCollection][Object]`

如果我们想实现泛型数组，我们也需要反射（不用泛型的话）。如果你这么写：
```java
public static Object[] badCopyOf(Object[] a, int newLength)//not useful
{
    Object[] new Array = new Object[newLength];
    Sstem.arraycopy(a,0,newArray,0,Math.min(a.length, newLength));
    return newArray;
}
```
然而这个代码在使用返回的数组时会遇到一个问题：它返回的是`(Object[])`，而且是创建的时候就是Object的数组，那么它无法转换成其它任何的数组。永远记住，一个其他类型数组临时转成`Object[]`，然后再转回来是可以的，但一个从开始就是`Object[]`就很糟糕了。为此需要java.lang.reflect包中Array类中的一些方法，比如newInstance。
```java
public static Object goodCopyof(Object a, int newLength)
{
    Class cl = a.getClass();
    if(!cl.isArray()) return null;
    Class componentType = cl.getComponentType();
    int length = Array.getLength(a);
    Object newArray = Array.newInstance(componentType, newLength);
    System.arraycopy(a, 0, newArray, 0, Math.min(length, newLength));
    return newArray;
}
//
String[] b = {"1","2","3"};
b = (String[]) goodCopyof(b,10);
```
建议有必要才使用Method对象，最好使用接口以及lambda。因为编译器对于invoke等等方法的检查非常弱，而且他们非常慢。
继承的设计技巧：
1. 将公共操作和域放在超类
2. 不要使用受保护的域
3. 用继承实现is-a关系
4. 除非所有继承的方法都有意义，否则不要使用继承
5. 在覆盖方法时，不要改变预期的行为
6. 使用多胎，而非类型信息
7. 不要过多地使用反射
