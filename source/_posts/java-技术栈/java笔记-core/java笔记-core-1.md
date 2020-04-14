---
title: java笔记-core-1
date: 2020-04-01 11:04:10
tags:
- java
- company
- java-core

categories:
- java
- book_notes

---
java笔记（简单）（东西很碎很杂，我就串一条大概的线）

```java
public class Classname
{
    public static void main(String[] args)
    {
        program statements
    }
}
```

<!--more-->

## 前期杂项
下划线和0b这种东西...有点像c++17，不过好像没有0b？翻了一下，对不起，都有，不是下划线是分号线，c++14
然后指数p
`0.125 = 2^-3 -> 0x1.0p-3`

unicode转义序列会在解析之前得到处理。想成是某种宏替换即可，注意注释也会生效。

这个UTF16给我看傻了。
找到了比较靠谱的https://blog.csdn.net/wusj3/article/details/88641084
具体说就是尽量不要用char

java不区分声明和定义（？其实我觉得c++后面也不分了，有统一的初始化要求，但是没有强制的区别....?）

const是Java保留但没有被使用。java必须用final定义常量。不过final在c++中是禁止派生

一般而言整数转浮点会丢精度，但是int转double却不会。不要强转bollean

`>>>可以用来用0填充高位，>>使用符号位填充。不存在<<<。即是说>>对于负数生成的结果可能会依赖于具体的实现（c++），java则不会。`

Java不使用逗号运算符。不过for里可以有。

## String
String没有提供用于修改字符串的方法。这一点很像lua了。
substring什么的都很像。

一定不要用==判断字符串是否相等！它只能确定两个字符串是否放在同一个位置上。因为虚拟机可以想象为把字符串放在公共的存储池中——共享。但是在java里这就像c++的未定义行为一样的发生随机间歇性bug。

length返回代码单元数量，charAt(n)返回位置n的的代码单元，codePointAt(index)返回码点，condePoints方法会更好用一些。
codePoints()将字符串的码点作为一个流返回，调用toArray将它们放在一个数组中

输入输出流有点复杂了就，应该说是一层套一层套一层
StringBuilder();
读取：Scanner in = new Scanner(System.in);
StreamTokenizer(...)
PrintWriter(
BufferedReader(
InputStreamReader(
System.out)))

printf中参数索引从1开始，一般的索引（指数组还都是从0开始）

---
## java数组

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
## java的类（和方法）
一个对象变量，并没有，实际，包含一个对象。
隐含的意味就是，不存在默认构造(指声明的时候就帮你初始化的操作)
系统会自动提供默认的构造器
局部变量不会自动地初始化为null
java的对象可以被看作c++的对象指针，即
`Date Birthday(java) = Date* birthday(c++)`

-------------------------------
### 更改器方法和访问器方法
在c++中，带有const后缀的方法就是访问器方法，默认就是更改器方法。java里这两种方法在语法上没有明显的区别
c++程序员最容易犯的错误就是忘记了new操作符
隐式参数可以不用this表示，直接用

-------------------------------
### 不要返回引用可变对象的访问器
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
注意上面的这个例子中，不要编写返回引用可变对象的访问器方法，这破坏了封装性。但是另一个方面，这里的返回值就直接是引用，这点值得注意。
如果需要返回一个这样的引用，应该使用它的clone（某种意义上，就是右值）

-----------------------------
### 方法
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
#### 关于调用
Java总是采用按值调用，方法不能修改传递给它的任何参数变量的内容。但是，但是对象引用的状态是可以改变的。
【说白了就是java对于对象来说用的都是一种类似引用的调用，具体应该就是池的问题，这是我猜的，但是要注意了：
* java对对象采用的是引用调用 这个说法是不对的。
比如交换两个参数，swap(a,b)，交换的却是两个拷贝。所以最好的理解就是创建了一个拷贝，这两个东西（a本身和a在方法里的拷贝）指向的对象是同一个对象，对象的状态改变，即使参数变量不再使用，原来的变量依然指向该改变过的对象。

------------------------
方法的签名包括方法名和参数类型，不包括返回类型【但是我们在泛型里面会看到在编译器里是认返回类型的

域会被自动初始化为默认值，域也称成员变量。

c++和java一样可以直接初始化实例域（成员变量）

c++和java一样可以在一个构造器里调用另一个构造器，不过可能需要一些using和初始化列表的配合，语法不太一样。java是this(...)。

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
### 继承
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
我们会在后面看到非常非常多的，关于把子类赋给超类然后进行一些操作的行为，主要就是Object和泛型。

-----------------
#### 调用方法
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
#### Object的几个常用方法
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
vector重载了`[]`，但是java没有重载，必须调用显式的方法。此外，c++的stl是值拷贝，java是引用（就是赋给另一个值的时候）。

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

## 反射
+ 在运行时分析类的能力
+ 在运行时查看对象，例如编写一个toString方法供所有类使用
+ 实现通用的数组操作代码
+ 利用Method对象，这个对象很像C++中的函数指针

有一个专门的Java类访问java为所有对象维护的信息，他就是Class
最常用的方法就是getName，会返回类的名字
静态方法forName获得类名对应的Class对象，注意提供异常的情况
如果T是java类型，T.class代表匹配的类对象
一个Class对象实际上表示的是一个类型，而这个类型未必是一种类，T.class的类型其实就是`Class<T>`
`class.getDeclaredConstructor().newInstance()`可以用来动态的创建一个类的实例，调用默认的构造器

getClass方法，有多态能力，运行时可以返回子类的类型信息，
.class是没有多态的，是静态解析的，编译时可以确定类型信息，虚拟机为每个类型管理一个.class对象

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