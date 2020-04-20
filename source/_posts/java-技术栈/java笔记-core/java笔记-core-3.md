---
title: java笔记-core-3
date: 2020-04-07 10:43:19
tags:
- java
- company
- java-core

categories:
- java
- book-notes

---

泛型、集合
<!--more-->

---

## 泛型

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

限定可以多接口，但是最多一个类。类限定必须是列表中的第一个

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

### 类型擦除的限制
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
### 泛型的继承
无论S和T有什么关系，通常`Pair<S>和Pair<T>`没有什么联系。
简单的例子，给Manager的Pair强转之后加一个Employee，这一对就很怪异了（当然是不可以的）
虽然数组可以这么做(指强转过去)，但是数组会在存储的时候抛出异常（指数组有特别的保护)
泛型可以扩展，扩展就可以转换，但是只能是`A<M>到B<M>`，而不能是`A<M>到A<E> or B<E>`。
可以将参数化类型转化为一个原始类型，之后会正常产生类型错误。

-----------------------------
### 通配符类型
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
## 集合
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

集合类的基本接口是Collection接口，里面其实有很多的方法。
```java
public interface Collection<E>
{
    boolean add(E element);
    Iterator<E> iterator();
    /**
     * iterator包含4种方法：
     * E next();
       boolean hasNextO;
       void remove0;
       default void forEachRemaining(Consumer<? super E> action);
     */
    int size()
    boolean isEmpty()
    boolean contains (Object obj)
    boolean containsAl1 (Col1ection<?> c)
    boolean equals (Object other)
    boolean addAll (Collection<? extends E> from)
    boolean remove(Object obj)
    boolean removeAl1 (Col1ection<?> c)
    void clear()
    boolean retainAll (Col1ection<?> c)
    Object[] toArray()
    <T> T[] toArray(T[] arrayToFill)
}
```
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
### 视图与包装器
视图是这样一种集合：keySet方法返回一个实现Set接口的类对象，这个类的方法对原映射进行操作。
将集合类对象中的数据重新映射到一个数据集合中，但是这个集合不是以物理上存在的对象实体，而是再映射，物理地址不变，访问数据的接口变了。
Arrays类的静态方法asList将返回一个包装了普通java数组的List包装器，这样就可以将数组直接传递给列表或集合参数的方法了。
注意返回的对象不是ArrayList。而是一个带有访问底层数组get和set方法的视图对象。
通常，视图有一些局限性，即只可以读、无法改变大小、支持删除而不支持插入。
返回空类型有的时候很有用。`Set<String> deepThought = Collections.emptySet()`
不可修改视图、同步视图、受查视图。

## 并发
贴一下Runnable：

>在实际工作中，我们很可能习惯性地选择Runnable或Thread之一直接使用，根本没在意二者的区别，但在面试中很多自以为是的菜货面试官会经常而且非常严肃的问出：请你解释下Runnable或Thread的区别？尤其是新手就容易上当，不知如何回答，就胡乱编一通。鄙人今天告诉你们这二者本身就没有本质区别，就是接口和类的区别
>Runnable的实现方式是实现其接口即可
>Thread的实现方式是继承其类
>Runnable接口支持多继承，但基本上用不到
>Thread实现了Runnable接口并进行了扩展，而Thread和Runnable的实质是实现的关系，不是同类东西，所以Runnable或Thread本身没有可比性。
>结论，Thread和Runnable的实质是继承关系，没有可比性。无论使用Runnable还是Thread，都会new Thread，然后执行run方法。用法上，如果有复杂的线程操作需求，那就选择继承Thread，如果只是简单的执行一个任务，那就实现runnable。

如果想将弹跳球代码放在一个独立的线程中，只需要实现一个类BallRunnable，然后把动画代码放在run方法中，如下：
```java
Runnable r = () -> {
    try
    {
        for(int i = 1; i <= STEPS; i++)
        {
            ball.move(comp.getBounds());
            comp.repaint();
            Thread.sleep(DELAY);
        }
    }catch(InterruptedException e)
    {
    }
};
Thread t = new Thread(r);
t.start();
```
>也可以通过Thread类的子类定义一个线程，不过这种方法已经不推荐了，应该将要并行的任务与运行机制解耦合。如果有很多任务，为每个任务创建独立线程付出的代价太大了，可以使用线程池来解决这个问题

>不要调用Thread类或Runnable对象的run方法，直接调用run方法，指挥执行同一个线程中的任务，而不会启动新线程。应当用Thread.start方法，这将创建一个执行run方法的新线程

### 中断线程
当线程中的run方法执行方法体中最后一条语句后，并经由执行return语句返回时，或者出现了在方法中没有捕获的异常时，线程将终止。
没有可以强制线程终止的方法，interrupt可以用来请求终止线程。
注意一个被阻塞的线程上调用interrupt方法时，阻塞调用会被InterruptedException异常中断。没有任何语言方面的需求要求被中断的线程应该终止。中断线程是引起它的注意，被中断的线程可以决定如何相应中断。有的线程如此重要以至于应该处理完异常后继续执行不理会中断；更普遍的情况是，线程是简单地将中断作为终止的请求。
```java
Runnable r = () -> {
    try
    {
        ...
        while(!Thread.currentThread().isInterrupted() && more work to do)
        {
            do more work;
        }
    }
    catch(InterruptedException e)
    {
        //thread was interrupted during sleep or wait
    }
    finally
    {
        cleanup, if required
    }
    //exiting the run method terminatees the thread
};
```
如果每次工作迭代之后调用sleep方法，isInterrupted检测既没必要也没用处。因为如果在中断状态置位时调用sleep，不会休眠，反而会清除中断并抛出InterruptedException。
isterrupted和isInterrupted不一样，前者清除该线程的中断状态，是静态方法，后者是实例方法。
### 线程状态
+ New
+ Runnable（可运行）
+ Blocked（被阻塞）
+ Waiting
+ Timed waiting（计时等待）
+ Terminated（被终止）
#### 可运行线程
一个可运行的线程可能正在运行，也可能没有运行。
抢占式调度和协作式调度...协作式是一个线程只有在调用yield方法或者被阻塞或等待时，线程才失去控制权。抢占式是时间片用完之后操作系统剥夺该线程运行权并给另一个线程运行机会。

#### 被阻塞线程和等待线程
当线程处于被阻塞或等待状态时，它暂时不活动。细节取决于怎样达到非活动状态的。
+ 当一个线程试图获取一个内部的对象锁，而该锁被其他线程持有，则线程进入阻塞状态。当所有其他线程释放该锁，并且线程调度器允许本线程持有它的时候，该线程将变成非阻塞状态
+ 当线程等待另一个线程通知调度器一个条件时，它自己进入等待状态。
+ 有几个方法有一个超时参数。调用它们导致线程进入计时等待状态。

#### 被终止的线程
+ 因为run方法正常退出而自然死亡
+ 因为一个没有捕获的异常终止

### 线程属性

#### 线程优先级
java中线程默认继承父线程的优先级，setPriority方法提高或降低任何一个线程的优先级。可以将优先级设置在MIN_PRIORITY(1)-MAX_PRIORITY(10)之间，NORM_PRIORITY定义为5
每当线程调度器有机会选择，会首先选择较高优先级的线程，但是线程优先级**高度依赖于系统**，不要讲程序构建行为的正确性依赖于优先级。

#### 守护线程
t.setDaemon(true);可以将线程转换为守护线程，唯一用途是为其他线程提供服务。计时线程就是一个例子，它定时地发送「计时器嘀嗒」信号给其他线程或清空过时的高速缓存项的线程。只剩下守护线程时，虚拟机就退出了。

#### 未捕获异常处理器（
线程的run不能抛出任何受查异常，非受查异常会导致线程终止。
但是不需要任何catch子句来处理可以被传播的异常。相反，在线程死亡之前，异常被传递到一个用于未捕获异常的处理器。
处理器必须实现一个Thread.UncaughtExceptionHandler接口的类，只有一个方法：
void uncaughtException(Thread t, Throwable e)
如果不为独立的线程安装处理器，此时的处理器就是该线程的ThreadGroup对象。
ThreadGroup类的uncaughtException方法做如下操作：
1. 如果有父线程组，那么父线程组的uncaughtException方法被调用
2. 否则，如果Thread.getDefaultExceptionHandler方法返回一个非空处理器，调用该处理器
3. 否则，如果Throwable是ThreadDeath的一个实例，什么都不做
4. 否则，线程名和Throwable的栈轨迹被输出到System.err上

### 同步
在大多数实际的多线程应用中，两个或两个以上的线程需要共享对同一数据的存取。根据各线程访问数据的顺序，可能会产生讹误的对象。这样的情况通常被称为竞争条件

#### 来看一个小例子
如何同步数据存取
模拟一个有若干账户的银行。随机地生成在这些账户之间转移钱款的交易。每一个账户有一个线程。
```java
public void transfer(int from, int to, double amount)
    //CAUTION: unsafe when called from multiple threads
{
    System.out.print(Thread.currentThread());
    accounts[from] -= amount;
    System.out.printf(" %10.2f from %d to %d", amount, from, to);
    accounts[to] += amount;
    System.out.printf(" Total Balance: %10.2f%n", getTotalBalance());
}
```
Runnable类的代码。run方法不断地从一个固定的银行账户取出钱款。迭代中，run方法随意选择一个目标账户和一个随机账户，调用bank对象的transfer方法，然后睡眠

```java
Runnable r = () ->{
    try
    {
        while(true)
        {
            int toAccount = (int) (bank.size() * Math.random());
            double amount = MAX_AMOUNT * Math.random();
            bank.transfer(fromAccount, toAccount, amount);
            Thread.sleep((int)(DELAY * Math.random()));
        }
    }
    catch(InterruptedException e)
    {
    }
}
```
假定两个线程同时执行指令：
```java
accounts[to] += amount;
```
问题在于这不是原子操作。该指令可能被处理如下
1. 将`accounts[to]` 加载到寄存器
2. 增加amount
3. 将结果写会`accounts[to]`
假定线程1执行步骤1和2，然后它被剥夺了运行权，然后线程2唤醒并修改了accounts数组中的同一项，然后线程1被唤醒并完成其第3步。
注意最后执行的第3步擦去了第二个线程所做的更新，于是总金额就不再正确了。
真正的问题即是transfer方法的执行过程中可能会被中断。如果能够确保线程在失去控制之前方法运行完成，那么银行账户对象的状态永远不会出现讹误。

#### 锁对象
有两种机制防止代码块受并发访问的干扰。java提供synchronized（同步）关键字达到这一目的，并且JavaSE5.0引入了ReentrantLock类。
ReentrantLock保护代码块的基本结构如下
```java
myLock.lock();
try
{
    critical section
}
finally
{
    myLock.unlock();//make sure the lock is unlocked even if an exception is thrown
}
```
这一结构确保任何时刻只有一个线程进入临界区。一旦一个线程封锁了锁对象，其他任何线程都无法通过lock语句。当其他线程调用lock时，它们被阻塞直到第一个线程释放锁对象。
*把解锁操作括在finally子句之内是至关重要的，如果在临界区的代码抛出异常，锁必须被释放，否则其他线程将永远阻塞*
如果使用锁，就不能使用带资源的try语句。（然后啰嗦了一些我听不懂的话，什么解锁方法名不是close。什么首部希望声明一个新变量，锁你可能使用多线程共享那个变量而不是新变量）
继续上面那个例子
```java
public class Bank
{
    private Lock bankLock = new ReentrantLock(); //ReentrantLock implements th Lock interface
    ...
    public void transfer(int from, int to, int amount)
    {
        bankLock.lock();
        try
        {
            System.out.print(Thread.currentThread());
            accounts[from] -= amount;
            System.out.printf(" %10.2f from %d to %d", amount, from, to);
            accounts[to] += amount;
            System.out.printf(" Total Balance: %10.2f%n", getTotalBalance());
        }
        finally
        {
            bankLock.unlock();
        }
    }
}
```
注意如果两个线程访问不同的Bank对象，每一个线程得到不同的锁对象，两个线程都不会发生阻塞。只有访问同一个Bank对象，锁会以串行方式提供服务。
锁是**可重入**的(Reentrant = 重入）（重入就是异步中中断或等待之后可以再重新进入该函数或代码继续运行，结果不受影响）.线程可以重复的获得已经持有的锁.锁保持一个持有计数来跟踪对lock方法的嵌套调用

*留心临界区的代码，不要因为异常抛出而跳出临界区。如果在临界区代码结束之前抛出了异常，finally子句将释放锁，但会使对象处于一种受损状态*

#### 条件对象
通常也叫条件变量。即线程进入临界区发现在某一条件满足后才能执行，需要一个条件对象来惯例那些已经获得一个锁却不能做有用工作的进程。
现在细化银行的模拟程序，避免选择没有足够资金的账户作为转出账户，但是注意了，不能：
```java
if(bank.getBalance(from) >= amount)
    //thread might be deactivated at this point
    bank.transfer(from, to, amount);
```
必须确保没有其他线程在本"检查余额与转账"活动之间修改余额。通过锁来保护"检查与转账"动作来做到这一点：
```java
public void tranfer(int from, int to, int amount)
{
    bankLock.lock();
    try
    {
        while(accounts[from] < amount)
        {
            //wait
            ...
        }
        //transfer funds
        ...
    }
    finally
    {
        bankLock.unlock();
    }
}
```
当账户中没有足够的余额时，我们该做什么呢？等待直到另一个线程向账户中注入了资金。但是这一线程刚刚获得了对bankLock的排他性访问，别的进程没有进行存款操作的机会。

一个锁对象可以有一个或多个相关的条件对象。你可以用newCondition方法获得一个条件对象。习惯上给每一个条件对象命名为可以反应它表达的条件的名字。例如这里就是余额充足：
```java
class Bank
{
    private Condition sufficientFunds;
    ...
    public Bank()
    {
        ...
        sufficientFunds = bankLock.newCondition();
    }
}
```
如果transfer方法发现余额不足，它调用
sufficientFunds.await();
(常用while套住)
当前线程现在被阻塞了，并放弃了锁
当另一个线程转账时，它应该调用
sufficientFunds.signalAll();
这一调用重新激活因为这一条件而等待的所有进程。当这些线程从等待集中移除时，再次成为可运行的，调度器再次激活它们。同时它们将试图重新进入该对象，一旦锁成为可用的，它们中的某个将从await调用返回，获得该锁并从被阻塞的地方继续执行。
至关重要的是需要某个其他线程调用signalAll方法。当一个线程await时它没有办法重新激活自身——只能由其他线程做这事。如果没有其他线程来重新激活等待的线程，那么它就永远不再运行了，这将导致「死锁」，如果所有其他线程被阻塞...所以当在对象的状态有利于等待线程的方向改变时调用signalAll
现在来看一下正确的程序：
```java
//synch/Bank.java

public class Bank
{
    private final double[] accounts;
    private Lock bankLock;
    private Condition sufficientFunds;
    /**
     * @param n the number of accounts
     * @param initialBalance the initial balance for each account
     */
     public Bank(int n,double initialBalance)
     {
         accounts = new double[n];
         Arrays.fill(accounts, initialBalance);
         bankLock = new ReentrantLock();
         sufficientFunds = bankLock.newCondition();
     }
     public void transfer(int from, int to, double amount) throws InterruptedException
     {
         bankLock.lock();
         try
         {
            while(accounts[from] < amount)
                sufficientFunds.await();
            System.out.print(Thread.currentThread());
            accounts[from] -= amount;
            System.out.printf(" %10.2f from %d to %d", amount, from, to);
            accounts[to] += amount;
            System.out.printf(" Total Balance: %10.2f%n", getTotalBalance());
            sufficientFunds.signalAll();
         }
         finally
         {
             bankLock.unlock();
         }
     }
     public double getTotalBalance()
     {
         bankLock.lock();
         try
         {
             double sum = 0;
             
             for(double a : accounts)
                sum += a;
            
            return sum;
         }
         finally
        {
            bankLock.unlock();
        }
     }
     public int size()
     {
         return accounts.length;
     }
}
```
想搞清楚上面说的乱七八糟的signalAll还是要靠signalAll的原理来解决。
简单一点，不牵扯bullshit的源码的话：signalAll方法，使得已经await的线程awaitThread能够有机会移入到同步队列中，当其他线程释放lock后使得线程awaitThread能够有机会获取lock，从而使得线程awaitThread能够从await方法中退出执行后续操作
至于释放之后锁竞争的顺序，要看lock的公平性来实现

#### synchronized关键字
+ 锁用来保护代码片段，任何时刻只能有一个线程执行被保护的代码
+ 锁可以管理试图进入被保护代码段的线程
+ 锁可以拥有一个或多个相关的条件对象
+ 每个条件对象管理那些已经进入被保护的代码段但还不能运行的线程
如果一个方法用synchronize关键字声明，那么对象的锁将保护整个方法，只有一个相关条件：wait，notifyAll/notify方法接触等待线程的阻塞状态。
存在一些局限，包括：
+ 不能中断一个正在试图获得锁的线程
+ 试图获得锁时不能设定超时
+ 每个锁仅有单一的条件可能是不够的。

那么代码中应该使用哪一种？Lock和Condition对象还是同步方法？
+ 最好既不用Lock/Condition也不实用synchronized关键字。许多情况下你可以使用java.util.concurrent包中的一种机制，它会为你自动处理所有而加锁。
+ 如果synchronized适合，那么请尽量使用它。
+ 如果特别需要Lock/Condition结构提供的独有特性时，才用它。

#### 同步阻塞
可以通过另一种机制获得锁：进入一个同步阻塞
```java
synchronized(obj)//this is the syntax for a synchornized block
{
    critical section
}
```
有时程序员使用一个对象的锁来实现额外的原子操作，这被称为「客户端锁定」
这种方法非常脆弱，不推荐使用。需要的话自己去查好了


#### 监视器理念
锁和条件是线程同步的强大工具，但是严格地讲，它们不是面向对象的。多年来，研究人眼努力寻找一个方法可以在不需要程序员考虑如何加锁的情况下就保证多线程的安全性。最成功的的方案之一就是监视器。
然而在下述的三个方面Java对象不同于监视器，从而使得线程的安全性下降

#### Volatile域
同步格言：「如果向一个变量写入值，而这个变量接下来可能会被另一个线程读取，或者，从一个变量读值，而这个变量可能是之前被另一个线程写入的，此时必须使用同步」

volatile关键字为实例域的同步访问提供了一种免锁机制。如果声明一个域为volatile，那么编译器和虚拟机就知道该域是可能被另一个线程并发更新的。

但是注意：Volatile变量不能提供原子性

#### final变量
除非使用锁或者volatile修饰符，否则无法从多个线程中安全地多去一个域。
还有一种情况可以安全地访问一个共享域，即这个域声明为final时。
```java
final Map<String, Double> accounts = new HashMap<>();
```
其他线程会在构造函数完成构造后才看到这个accounts变量。
如果不用final，那么可能看到的是null，而不是新构造的hashmap
当然，操作仍然不安全

#### 原子性
假设对共享变量除了赋值之外无其他操作，那么可以将这些共享变量声明为volatile
java.util.concurrent.atomic包中有很多类使用了高效的机器级指令来保证其他操作的原子性
```java
public static AtomicLong nextNumber = new AtomicLong();
long id = nextNumber.incrementAndGet();
```
如果希望完成更复杂的更新，就必须使用compareAndSet方法。
例如，假设希望跟踪不同线程观察的最大值，下面的代码不可行：
```java
public static AtomicLong largest = new AtomicLong();
largest.set(Math.max(largest.get(), observed));//Error--race condition!
```
这个更新不是原子的，事实上，应当在一个循环中计算新值和使用CAS
```java
do{
    oldValue = largest.get();
    newValue = Math.max(oldValue, observed);
} while(!largest.compareAndSet(oldValue, newValue));
```
如果另一个线程也在更新largest，就可能阻止这个线程更新。这样一来，CAS会返回false，而不会设置新值。在这种情况下，循环会更次尝试读取更新后的值并尝试修改。
CAS比锁要快得多。
Java SE 8中，不再需要编写这样的循环样板代码。
```java
largest.updateAndGet(x -> Math.max(x, observed));
//或
largest.accumulateAndGet(observed, Math::max);
```
如果有大量线程要访问相同的原子值，性能会大幅下降，因为乐观更新需要太多次重试。Java SE 8提供了LongAdder和LongAccumulator类来解决这个问题。LongAdder包括多个变量，总和为当前值。即可以有多个线程更新不同的加数。

#### 死锁
锁和条件不能解决多线程中的所有问题。考虑：
    账户1:$200
    账户2:$300
    线程1:从账户1转移$300到账户2
    线程2:从账户2转移$400到账户1
有可能会因为每一个线程要等待更多的钱款存入而导致所有线程都被阻塞，这样的状态成为死锁。
让第i个线程负责向第i个账户存钱而不是取钱可能会使所有线程集中到一个账户上，每个账户都视图取出大于余额的钱于是死锁。
signalAll转换为signal最终会死锁。signal仅对一个线程解锁，如果它不能继续运行，所有的线程可能被阻塞
比如
    账户1:1990
    其他账户:990
    线程1:从账户1转995到2
    所有其他:从它们的账户转995到另一家
    ---
    线程1继续执行，运行后如下：
    账户1:995
    账户2:1985
    其他:990
    ---
    线程1调用signal，假定选择线程3，该线程被唤醒发现它没有足够的金额，于是它继续await，线程1仍在运行，随机的产生新的交易，例如：
    线程1:从线程1转997到账户2
    现在线程1也await了，所有的线程都被阻塞，系统死锁。
遗憾的是，java编程语言中没有任何东西可以避免或打破这种死锁。
你必须确保程序保证没有死锁。

#### 线程局部变量
在使用线程不安全的类时，结果可能会很混乱，可以使用同步或者在需要时构造一个局部SimpleDateFormat对象，不过这很浪费或开销很大。
要为每个线程构造实例，可以使用：
```java
public static final ThreadLocal<SimpleDateFormat> dateFormat = 
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
```

#### 锁测试和超时
tryLock方法试图申请一个锁，成功获得锁就返回true，否则返回false，而且线程可以立即离开去做其他事情。
if(mylock.tryLock(100,TimeUnit.MILLISECONDS))...
lock方法不能被中断，如果一个线程在等待锁时被中断，中断线程在获得锁时一直被阻塞。如果死锁，那么lock方法就无法终止
然而如果使用带超时参数的tryLock，那么线程如果在等待期间被中断，将抛出InterruptedException异常，这是非常有用的特性，允许程序打破死锁

#### 读写锁
如果很多线程从一个数据结构读取数据而很少修改其中的数据的话，ReentrantReadWriteLock类是非常有用的。允许读者线程共享访问是合适的。当然，写者线程必须是互斥访问的。
```java
private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
private Lock readLock = rwl.readLock();
private Lock writeLock = rwl.writeLock();

public double getTotalBalance()
{
    readLock.lock();
    try{...}
    finally{ readLock.unlock(); }
}

public void transfer(...)
{
    writeLock.lock();
    try{...}
    finally{writeLock.unlock();}
}
```
#### 为什么弃用stop和suspend
初始的Java版本定义了一个stop方法用来终止一个线程，以及一个suspend方法用来阻塞一个线程直至另一个线程调用resume。stop和suspend方法有一些共同点：都试图控制一个给定线程的行为
stop终止所有未结束的方法，包括run方法。当线程被终止，立即释放被他锁住所有对象的锁，这会导致对象处于不一致的状态。比如，TransferThread在从一个账户向另一个账户转账的过程中被终止，钱款已经转出却还没有转入目标账户。
当线程要终止另一个线程时，无法知道什么时候调用stop方法是安全的，什么时候导致对象被破坏。

如果用suspend方法挂起一个持有一个锁的线程，那么，该锁在恢复之前是不可用的。如果调用suspend方法的线程试图获得同一个锁，那么程序死锁：被挂起的等恢复，将其挂起的在等待获得（自己搞自己）。
图形界面常常会出现这种情况。

安全地挂起线程，引入变量suspendRequested并在run方法的某个安全的地方测试它。
### 阻塞队列
事实上，实际编程来说应尽可能远离底层结构。
对于许多线程问题，可以通过一个或多个队列优雅且安全的方式将其形式化。生产者线程向队列插入元素，消费者线程则取出它们。
当试图向队列添加元素而队列已满或是想从队列里移出元素而队列为空的时候，阻塞队列导致线程阻塞。在协调多个线程之间的合作时，阻塞队列是一个有用的工具。

```java
import java.io.*;
import java.util.*;
import java.util.concurrent.*;

//this is a concurrent java app which can print the keyword you want to search in multiple files.
public class t9 {
    private static final int FILE_QUEUE_SIZE = 10;
    private static final int SEARCH_THREADS = 100;
    private static final File DUMMY = new File("");
    private static BlockingQueue<File> queue = new ArrayBlockingQueue<>(FILE_QUEUE_SIZE);
    public static void main(String[] args)
    {
        try(Scanner in = new Scanner(System.in))
        {
            System.out.print("Enter bas dircetory (e.g. /opt/jdk1.8.0/src):");
            String directory = in.nextLine();
            System.out.print("Enter keyword(e.g. volatile) ");
            String keyword = in.nextLine();

            Runnable enumerator = () -> {
                try{
                    enumerate(new File(directory));
                    queue.put(DUMMY);
                }
                catch(InterruptedException e)
                {
                }
            };

            new Thread(enumerator).start();
            for(int i = 1; i <= SEARCH_THREADS; i++){
                Runnable searcher = () -> {
                    try
                    {
                        boolean done = false;
                        while(!done)
                        {
                            File file = queue.take();
                            if(file == DUMMY)
                            {
                                queue.put(file);
                                done = true;
                            }
                            else search(file, keyword);
                        }
                    }
                    catch(IOException e)
                    {
                        e.printStackTrace();
                    }
                    catch(InterruptedException e)
                    {
                    }
                };
                new Thread(searcher).start();
            }
        }
    }
    
    /**
     * Recursively enumerates all files in a given directory and its subdirectories.
     * @param directory the directory in which to start
     */
    public static void enumerate(File directory) throws InterruptedException
    {
        File[] files = directory.listFiles();
        for(File file :files)
        {
            if(file.isDirectory()) enumerate(file);
            else queue.put(file);
        }
    }

    /**
     * Searches a file for a given keyword and prints all matching lines;
     * @param flie the file to search
     * @param keyword the keyword search for
     */
    public static void search(File file, String keyword) throws IOException
    {
        try (Scanner in = new Scanner(file, "UTF-8"))
        {
            int lineNumber = 0;
            while (in.hasNextLine())
            {
                lineNumber++;
                String line = in.nextLine();
                if(line.contains(keyword))
                System.out.printf("%s:%d:%s%n",file.getPath(),lineNumber,line);
            }
        }
    }
}

```
枚举所有文件放到一个阻塞队列中，这个操作很快，如果没有上限的话很快就包含了所有找到的文件。
我们启动了大量搜索线程。每个搜索线程从队列中取出一个文件，打开并打印带关键字的行。然后取下一个。
我们用一个虚拟对象来终止。
注意不需要显式的线程同步，队列数据结构就是我们的同步机制。
### 线程安全的集合
【这里你可以熟悉这些集合，最重要的是熟悉函数式接口，熟悉lambda。
这比学习并发集合本身的意义大得多
#### 高效的映射、集合队列
用Concurrent前缀修饰的HashMap、SkipListMap、SkipListSet和LinkedQueue
集合返回弱一致性的迭代器。即迭代器不一定能反映出它们被构造之后的所有的修改，但他们不会将同一个值返回两次。
#### 映射条目的原子更新
注意到线程安全的数据结构会允许非线程安全的操作。但注意，像get之后put这种操作中，数据结构不会被破坏，但操作序列不是原子的，所以结果不可预知。
```java
do
{
    oldValue = map.get(word);
    newValue = oldValue == null ? 1 :oldValue +1;
} while (!map.replace(word, oldValue, newValue));
//---------------------
map.putIfAbsent(word,new LongAdder());
map.get(word).increment();
//---------------------
map.putIfAbsent(word,new LongAdder()).increment();
//-----------------------
map.compute(word, (k,v) -> v == null ? 1 : v + 1);
//-----------------------
map.merge(word, 1L, (eV, nV) -> eV +nV)//existingValue newValue
//-----------------------
map.merge(word, 1L, Long::sum);
//merge就是Java8提供的函数式接口
```
#### 对并发散列映射的批操作
批操作无须冻结当前映射的快照，使用时需要将结果看作映射状态的一个近似
+ 搜索(search)为每个键或值提供一个函数，直到生成一个非null的结果
+ 规约(reduce)组合所有键或值
+ forEach
每个操作都有4个版本：处理键(Keys)、处理值(Values)、处理键和值、处理Map.Entry对象(Entries)
search的四个版本
```java
U searchKeys(long threshold, BitFunction<? super K, ? extends U> f)
U searchValues(long threshold, BitFunction<? super K, ? extends U> f)
U search(long threshold, BitFunction<? super K,? super V, ? extends U> f)
U searchEntries(long threshold, BitFunction<Map.Entry<K,V>, ? extends U> f)
```
假设我们想找第一个出现次数超过1000次的单词，需要搜索键和值：
```java
String result = map.search(threshold, (k, v) -> v > 1000 ? k : null);
```
forEach方法有两种形式：第一个只为各个映射条目提供一个消费者函数，如：
```java
map.forEach(threshold,
    (k, v) -> System.out.println(k + "->" + v));
```
第二种还有一个转换器函数，这个函数要先提供
```java
map.forEach(threshold,
    (k, v) -> k + "->" + v,
    System.out::println);
```
转换器可以作为一个过滤器，转换器返回null的就会被跳过了，例如只打印有大值的条目：
```java
map.forEach(threshold,
    (k, v) -> v > 1000 ? k + " -> " + v : null,
    System.out::println);
```
reduce操作用一个累加函数组合其输入，例如可以如下计算所有值的和：
```java
Long sum = map.reduceValues(threshold, Long::sum);
```
提供转换器也是可以的，可以如下计算最长的键的长度
```java
Integer maxlength = map.reduceKeys(threshold,
    String::length,
    Integer::max);
```
#### 并发集视图
没有提供对应的HashSet，你应当用HashMap的newKeySet方法生成一个`Set<K>`,或者用包含默认值的keySet方法，这样新的集也可以增加元素

#### 写数组的拷贝
CopyOnWriteArrayList是线程安全的集合

#### 并行数组操作
Arrays类提供了大量并行化的操作，比如parallelSort排序、parallelSetAll方法用函数计算的值填充数组（接收元素索引）
parallelPrefix，会用对应一个给定结合操作的前缀的累加结果替换各个元素，比如
```java
//[1,2,3,4....] 和 X
Arrays.parallelPrefix(values,(x,y) -> x*y)
//[1,1X2,1X2X3,1X2X3X4],注意每次左边的x都是新的前面的值
```
#### 较早的线程安全集合
Vector和Hahstable，已经不用了。
注意任何集合类都可以通过同步包装器变成线程安全的，但最好是使用concurrent包中定义的集合而不是同步包装器中的。

### Callable和Future
Runnable封装一个异步运行的任务，可以把它想象成一个没有参数和返回值的异步方法。
Callable与Runnable类似，但是有返回值，Callable接口是参数化的类型，只有一个方法call
```java
public interface Callable<V>
{
    V call() throws Exception;
}
```
Future保存异步计算的结果，可以启动一个计算将Future对象交给某个线程，然后忘掉他。Future对象的所有者可以在结果计算好之前就获得它。（人如其名
```java
public interface Future<V>
{
    V get() throws ...;
    V get(long timeout, TimeUnit unit) throws ...;
    void cancel (boolean mayInterrupt);
    boolean isCancelled();
    boolean isDone();
}
```
第一个get调用被阻塞直到计算完成，如果在计算完成之前第二个方法的调用超时，抛出TimeoutException异常，如果运行该计算的线程被中断，两个方法都将抛出InterruptedException。如果计算已经完成，那么get方法立刻返回
FutureTask 包装器是一种非常便利的机制， 可将 Callable转换成 Future 和 Runnable, 它同时实现二者的接口。
```java
Callable<Integer> myComputation = . . .;
FutureTask<Integer> task = new FutureTask<Integer>(myComputation);
Thread t = new Thread(task); // it's a Runnable
t.start()；
Integer result = task.get()；// it's a Future
```
程序清单 14-10 中的程序使用了这些概念。这个程序与前面那个寻找包含指定关键字的文件的例子相似。然而，现在我们仅仅计算匹配的文件数目。因此，我们有了一个需要长时间运行的任务，它产生一个整数值，一个` Callable<Integer> `的例子。
```java
class MatchCounter implements Callable<Integer〉
{
    public MatchCounter(File directory, String keyword) { ... }
    public Integer call() { . . . } // returns the number of matching files
}
```
然后我们利用 MatchCounter 创建一个 FutureTask 对象， 并用来启动一个线程。
```java
MatchCounter counter = new MatchCounter(new File(directory), keyword);
FutureTask<Integer> task = new FutureTask<Integer>(counter);
Thread t = new Thread(task);
t.start();
```
最后，我们打印结果。
```java
System.out.println(task.get() + " matching files.") ;
```
当然， 对 get 的调用会发生阻塞， 直到有可获得的结果为止。
在 call 方法内部， 使用相同的递归机制。 对于每一个子目录， 我们产生一个新的MatchCounter 并为它启动一个线程。此外， 把 FutureTask 对象隐藏在`ArrayList<Future<Integer>>`中。最后， 把所有结果加起来：
```java
for (Future<Integer> result : results)
    count += result.get();
```
每一次对 get 的调用都会发生阻塞直到结果可获得为止。 当然，线程是并行运行的， 因此， 很可能在大致相同的时刻所有的结果都可获得

```java
//--这里欠一代码
```

### 执行器
构建一个新的线程是有一定代价的，因为涉及和操作系统的交互。如果需要创建大量生命期很短的线程，应该使用线程池(thread pool)。一个线程池中包含许多准备运行的空闲线程。将Runnable对象交给线程池，就会有一个线程调用run方法，run方法退出时线程不会死亡而是继续在池中准备为下一个请求提供服务。
另一个使用线程池的理由是减少并发线程的数目。创建大量线程会大大降低性能甚至使虚拟机崩溃。
执行器(Executor)类有许多静态工厂方法用来构建线程池
#### 线程池
newCachedThreadPool必要时创建新线程，空闲线程保留60s
newFixedThreadPool是固定大小的线程池，空闲线程保留
newSingleThreadExecutor是一个退化了的大小为1的线程池，顺序执行每一个提交的任务
可用下面的方法之一将一个Runnable对象或Callable对象提交给ExecutorService：
```java
Future<?> submit(Runnable task)
Future<T> submit(Runnable task,T result)
Future<T> submit(Callable<T> task)
```
1. 调用Executors类中静态的方法newCachedThreadPool或newFixedThreadPool
2. 调用submit提交Runnable或Callable对象
3. 想要取消一个任务或如果提交Callable对象，那就要保存好返回的Future对象
4. 不再提交任何任务时，调用shutdown
```java
//这里有个线程程序
```

#### 预定执行
ScheduledExecutorService接口具有为预定执行或重复执行任务而设计的方法。它是一种允许使用线程池机制的java.util.Timer的泛化。
具体见文档
#### 控制任务组
使用执行器有更实际的原因，控制一组相关任务。例如，可以在执行器中使用shutdownNow方法取消所有的任务
invokeAny方法提交所有对象到一个Callable对象的集合中，并返回某个已经完成了的任务的结果。无法知道返回的究竟是哪个任务的结果。比如你要解决RSA密码，只要一个答案就可以了。
invokeAll方法提交所有对象到一个Callable对象的集合中，并返回一个Future对象的列表，代表所有任务的解决方案。当结构可获得时，可以像下面这样对结果进行处理
```java
List<Callable<T>> tasks = ...;
List<Future<T>> results = executor.invokeAll(tasks);
for (Future<T> result :results)
    processFurther(result.get());
```
这个方法的缺点是如果第一个任务恰巧花去了很多时间，则可能不得不进行等待。可以用ExecutorCompletionService进行排列。
```java
ExecutorCompletionService<T> service = new ExecutorCompletionService<>(executor);
for (Callable<T> task : tasks) service.submit(task);
```

#### Fork-Join 框架
有些应用使用了大量线程，但其中大多数都是空闲的。比如一个Web服务器可能会为每个连接分别使用一个线程。另外一些应用可能对每个处理器内核分别使用一个线程，来完成计算密集型任务，如图像或视频处理。fork-join框架专门用来支持这类应用。
```java

```
invokeAll方法接收到很多任务并阻塞，直到所有这些任务都已经完成。join方法将生成结果
在后台，fork-join给出一种叫工作密取(work stealing)的方法来平衡可用线程的工作负载，每个工作线程都有一个Deque(双向队列)来完成任务。

#### 可完成Future
### 同步器
#### 信号量
#### 倒计时门栓
#### 障栅
#### 交换器
#### 同步队列
### 线程与Swing