---
title: effectivecpp笔记-2
date: 2020-02-01 12:32:59
tags:
- old c++
- home

categories:
- c++
---

写着写着就成了摘抄了，这书确实...内容充实
资源管理与设计、声明
<!--more-->

---


# 资源管理
所谓资源就是，一旦用了就必须归还给系统之物。
内存只是众多资源之一，他们还有：文件描述符(finle descriptors)、互斥锁、图形界面中的某些东西、数据库连接、网络sockets。
## 13:以对象管理资源
+ 依赖「某函数总是会执行其delete语句」是行不通的

### RAII
>Resource Acquisition Is Initialization

+ 获得资源后立刻放进管理对象内。
以对象管理资源的思想经常被称为「资源取得的时机便是初始化时机」(RAII)，因为我们几乎总是在获得一笔资源后于同一语句内以它初始化某个管理对象。

+ 管理对象运用析构函数确保资源被释放

这里提了一些auto_ptr的某些使用问题，不过因为被废弃了所以尽量少用吧

【？96年居然就有shared_ptr了，不愧是TR1】

+ auto_ptr和tr1::shared_ptr在析构函数里做delete而不是delete[]，所以在动态array中不要用智能指针。

## 14:在资源管理类中小心copying行为

+ 复制RAII对象必须一并复制它所管理的资源

+ 普遍而常见的RAII class copying行为是：一致copying、施行引用计数法。
在引用计数法中作者举例需要指定shared_ptr以防止对象在计数为0时不要被删掉：
```c++
void lock(Mutex* pm);
void unlock(Mutex* pm);
//...
class Lock{
public:
    explicit Lock(Mutex* pm):mutexPtr(pm,unlock)
    {
        lock(mutexPtr.get());
    }
private:
    std::tr1::shared_ptr<Mutex> mutexPtr;
};
```

+ 深拷贝

+ 转移底部资源的拥有权问题。

## 15:在资源管理类中提供对原始资源的访问
关于tr1::shared_ptr的显式和隐式转换

+ .get()是显式转换（返回原始指针的复件）

+ 重载了指针取值操作符，以方便隐式转换至底部原始指针。

问题在于APIs往往要求访问Raw resources（原始资源），所以每一个RAII class应该提供相应方法：
### 关于RAII class
对于必须取得RAII对象内原始资源，提供隐式转换函数时可能出现的问题：
```c++
FontHandle getFont();//     C API，简化掉参数
void releaseFont(FontHandle fh)//    同一组C API
class Font{
public:
    explicit Font(FontHandle fh):f(fh)
    {}//     采用pass-by-value，因为C API 这样做。
    FontHandle get() const{ return f; }//显式
    //为防止大量想使用api就必须调用get，提供隐式
    //如：
    /*
    void changeFontSize(FontHandle f,int newSize);//C API
    Font f(getFont());
    int newFontSize();
    changeFontSize(f.get(),newFontSize);
    */
    operator FontHandle() const { return f; }
    /*
    Font f(getFont());
    int newFontSize();
    changeFontSize(f,newFontSize);
    */
private:
    FontHandle f;
};
```
问题是客户可能会在需要Font时意外创建一个FontHandle：
```c++
Font f1(getFont());
FontHandle f2 = f1; //原意是拷贝一个Font对象
                    //却反而将f1隐式转换为底部的
                    //FontHandle然后才复制它
```
即FontHandle由Font对象f1管理，但FontHandley也可以通过直接使用f2取得。那几乎不会有好下场。例如f1销毁，Font释放，但f2此时却是dangle（悬挂、虚吊）的

## 16:成对使用new和delete时要采取相同形式

### 请看错例
```c++
std::string* stringArray = new std::string[100];
...
delete stringArray;
```
行为不明确：100个对象中的99个不太可能被适当删除。

当你使用new/delete时会发生什么
1. 内存被分配出来
2. 针对此内存会有一个（或更多）构造函数被调用

1. 针对此内存会有一个（或更多）析构函数被调用
2. 释放内存。

问题在于：即将被删除的内存内究竟有多少对象？

这个问题指向的是「 删除的指针所指是数组吗 」

这里对于typedef和using等动作尤为重要。

## 17:以独立语句将新建的对象置入对象指针
考虑如下语句：
```c++
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw,int priority);
processWidget(std::tr1::shared_ptr<Widget>(new Widget),priority())
```
c++神奇的地方又出现了：
无有定义的编译器优化顺序：即有可能会是这样的：
1. 执行new Widget
2. 调用priority()
3. 调用tr1::shared_ptr构造函数
真如此，那么一旦在priority调用时导致异常，遗失了Widget返回的指针，会怎样？（为什么会遗漏？）
内存泄漏。使得，因为在资源被创建和资源被转化为资源管理对象两个时间点之间可能发生异常干扰
避免的方法很简单，把两个语句分开。
*注意c++17关于优化表达式顺序只是禁止交织，但是并未定义参数函数的调用顺序，他们依然是不确定的*

# 设计与声明
关于如何设计和声明一个优秀的接口。
关于最频繁出现的几个问题。

## 18:让接口很容易被正确的使用，很难被错误的使用。
一个准则：尽量使你的type的行为与内置type一致。

任何接口如果要求客户必须记得做某些事情，就有着「不正确使用」的倾向。
阻止这种误用倾向的办法有：建立新类型、限制类型上的操作、束缚对象值、消除客户的资源管理责任。

shared_ptr是如此好用，以致于我们必须考虑它的成本：比原始指针大且慢、使用辅助动态内存、并在多线程时受线程同步化的额外开销。
优势却非常多：消除「cross-DLL problem」的问题；带有删除器，避免各种资源泄漏，并在接口设计上非常优秀。

## 19:设计class犹如设计type
其实上条基本上提到了这点，但是我们要强调这种相似性，以致于为了设计高效的class，你的回答影响到设计规范的倾斜：
1. 新type的对象如何创建销毁？
2. 对象的初始化和对象的赋值该有什么样的区别？
构造函数和赋值操作符的行为和差异
3. 对象若被passed by value，意味着什么？
拷贝构造函数。
4. 什么是新type的合法值？
约束条件及成员函数（特别是构造函数、赋值操作符、所谓的setter函数）必须错误检查、抛出异常、函数异常说明(specifications)
5. type需要配合继承图系吗？
受父类的设计的束缚，尤其是virtual或non-virtual的影响
允许其他class继承，会影响到声明到函数（尤其是析构）是否为虚
6. 新的type需要什么样的转换？
7. 什么操作符对新type是合理的？
8. 什么样的默认函数应该驳回？
必须声明为private的，指
9. 谁该用新type成员？
关于成员public、protected、private的选择。
关于函数、类是否是friend以及嵌套的问题。
10. 什么是新type的「未声明接口」
指我们的type对效率、异常安全性以及资源运用提供何种保证？这些保证将加上相应的约束条件
11. 新type有多么一般化？
是否需要直接用template？
12. 你真的需要这个新type吗？

## 20:宁以pass-by-reference-to-const替换pass-by-value
我们这里避开移动构造函数也许是比较好的选择。
因为这里作者罗列的问题都被完美转发右值引用解决了。我们应该去看看在mordenc++里他有什么新的见解。这里我们单纯的来看这个const T&的神奇之处。
+ 效率高：无复制存在，回避了构造和析构
+ 保护好：是const，无法被更改。
+ 避免slicing（对象切割）问题
指继承类对象以传值方式传递并被视为一个基类，基类的拷贝构造函数被调用此时使之行为像继承类对象的特质就全被切割掉了。
还是看代码：
```c++
class Window{
    ...
};
class WindowWithScrollBars: public Window{
    ...
};
void printNameAAndDisplay(Window w)
{
    std::cout<< w.name();
    w.display();
}
```
这里的w会被构造成一个Window对象——就算传进去的是WindowWithScrollBars。
如果是个`const Window& w`就会表现出传进去的类型。

窥视c++编译器的底层，会发现references往往以指针实现出来。所以如果可以选择pass-by-value——尤其是哪些内置类型、stl、函数对象——就尽量使用pass-by-value

## 21:必须返回对象时，不要尝试返回一个reference
>一旦程序员领悟了pass-by-value的效率牵涉，往往进化为十字军战士，一心一意根除pass-by-value带来的种种邪恶。在追求reference的纯度中，他们一定会犯下一个致命错误：开始传递一些reference指向并不存在的对象。
```c++
class Rational{
public:
    Rational(int numerator = 0,int denominator = 1);//在24中说明不声明explicit的原因
    ...
private:
    int n,d;//分子、分母
    friend const Rational operator* (const Rational& lhs,const Rational& rhs);
};
```
你发现你传了值，于是想改成引用，但是你不能凭空变出对象，于是：
```c++
const Rational& operator* (const Rational& lhs,const Rational& rhs)
{
    Rational result（lhs.n * rhs.n,lhs.d * rhs.d）;
    return result;
}
```
emmm，用现在的话来说：你返回的这个值是将亡值。问题是你返回了一个引用。
成了，未定义。所有指向局部对象的引用下场都不会太好。
于是你想了想，考虑在heap里构造对象。
```c++
const Rational& operator* (const Rational& lhs,const Rational& rhs)
{
    Rational* result = new Ratiol(lhs.n * rhs.n, lhs.d * rhs.d);
    return *result;
}
```
你还是用了一个构造函数的调用。然后你发现了新的问题：谁该为你new的对象进行delete？
而且你发现这种delete是困难的：
```c++
Rational w,x,y,z;
w = x * y * z;
//operator*(operator*(x, y), z)
```
连续调用了两次operator\*，两次new，你需要两个delete。但是这时候你已经没有合理的办法取得operator*返回的reference背后隐藏的那个指针了。

你想起我们最初的目标是避免构造函数的多余消耗，你又有了主意。
还有办法可以避免构造函数:
```c++
const Rational& opertator* (const Rational& lhs,const Rationaal* rhs)
{
    static Rational result;
    result = ... ;//lhs * rhs，存进去。
    return result;
}
```
就像所有static的设计一样，我们会立刻产生对多线程安全性的疑虑。
更深层的瑕疵如下：
```c++
bool operator==(const Rational& lhs,
                const Rational& rhs);
Rational a,b,c,d;
...
if((a * b) == (c * d)) {
    //相等动作
}else {
    //不等动作
}
```
不用想了，if永为真。
因为他其实是这样的：
```c++
if(operator==(operator*(a, b), operator*(c, d)))
```
于是你又想到static array
行了别想了。array需要一个n，n要定义多大？还需要将值放进array内，赋值需要成本难道不高吗？

**当你必须在「返回一个reference和返回一个object中进行抉择时，选择正确的那个」**

让编译器厂商鞠躬尽瘁去吧。

## 22:将成员变量声明为private
### 不是public的原因
+ 语法一致性
不用记是用`.a`还是`.a()`
+ 用函数可以对变量又更精确的控制
不准、只读、读写甚至只写访问的控制
```c++
class AccessLevels{
public:
    ...
    int getReadOnly() const { return readOnly; }
    void setReadWrite(int value) { readWrite = value; }
    int getReadWrite() const { return readWrite; }
    void setWriteOnly(int value) { writeOnly = value;}
private:
    int noAAccess;
    int readOnly;
    int readWrite;
    int writeOnly;
};
```
+ 封装
```c++
class SpeedDateCollection {
    ...
public:
    void addValue(int speed);
    double averageSoFar() const;
    ...
};
```
考虑averageSoFar的实现。一种做法是在class内设计一个成员变量，记录至今以来的速度平均值，每次调用只需返回就好。另一种做法是每次调用时重新计算，此时有权调取每一个速度值。

第一种会使得每一个对象变大，因为你必须为用来存放平均值、累计总量、数据点数...的每个成员分配空间，然而函数却非常高效。后者则相反。
但我们但重点是，你封装了它，你可以替换不同的实现方式，客户最多只需重新编译（甚至不需）

### 一个准则
成员变量的封装性与「成员变量的内容改变时所破坏的代码数量」成反比。
是的，protected不比public好到哪去，所有使用它的derived class都会被破坏。
所以只有private（封装）和其他（不封装）。

## 23:宁以non-member、non-friend替换member函数
### 直接来例子
想象有个class用来表示网页浏览器。这样的class可能提供的众多函数中，有一些用来清除下载元素高速缓存区、请出访问过的URLs的历史记录、以及移除系统中的所有cookies：
```c++
class WebBrowser {
    public:
    ...
    void clearCache( );
    void clearHistory( );
    void removeCookies( );
    ...
};
//许多用户想一起执行他们，于是有：
class WebBrowser {
    public:
    ...
    void clearEverything( );//调用上面那仨
    ...
};
```
当然，也可以由一个由non-member函数调用适当的member函数而提供出来：
```c++
void clearBrowser(WebBrowser& wb)
{
    wb.clearCache();
    wb.clearHistory();
    wb.removeCookies( );
}
```
接下来我们讨论哪一个比较好。
### 一个直观的误解
面向对象的守则要求，数据以及操作数据的那些函数应该被捆绑在一块，这意味着member函数更好。不幸的是这个建议不正确。
与直观相反，成员函数clearEverything带来的封装性比clearBroser低。

### 关于封装：
如果某些东西被封装，那它就不再可见。愈多东西被封装，愈少的人看到，我们就有愈大的弹性去变化他。这就是我们推崇封装的原因：它使我们能够改变事物而只影响有限客户。

### 铺垫结束
能够访问private成员变量的函数只有class的member函数加上friend函数。
回过头来再看，如果一个non-member、non-friend函数和member函数提供相同机能，答案就很明显了，后者是输家。
有两点事值得注意：
+ 这个论述只适用于non-member、non-friend函数
+ 只因在意封装性而让函数「成为类的非成员」并不意味它「不可以是另一个类的成员」

### 关于...namespace
c++中，比较自然的做法是成为non-member并且要位于同一个namespace内。

一个像WebBrowser这样的class可能有大量的便利函数，通常客户只对其中一些感兴趣。分离的最直接办法是将书签相关的函数声明于一个头文件，将cookie相关函数声明与另一个头文件。
```c++
//头文件webbrower.h
namespace WebBrowserStuff{
class WebBrowser { ... };
    ...//核心机能，例如所有客户都需要的non-member函数
}

//头文件 webbrowserbookmarks.h
namespace WebBrowserStuff{
    ...//与书签相关的便利函数
}

//头文件 webbrowsercookies.h
namespace WebBrowserStuff{
    ...//与cookies相关的便利函数
}
```
另一个non-member non-friend函数的好处体现了出来：所有便利函数在多个头文件中却隶属同一命名空间，客户想用什么就声明什么头，同样也更方便扩展——只需添加更多函数到此命名空间即可（在新建的头文件里）

class可以派生，但问题在于派生类无法访问基类中被封装的成员，是一种...次级的身份。另外不是所有类都适合做基类。

## 24:如果所有参数都需要类型转换，声明non-member函数
这条有点长，我们一点一点来看怎么回事。

>令class支持隐式类型转换是个坏主意。

当然这条规则有其例外，最常见的例外是在建立数值类型时。假设你设计一个类用来表现有理数，那么允许整数隐式转换为有理数似乎非常合理。
```c++
class Rational {
public:
    Rational(int numberator = 0,int denominator = 1);//构造函数刻意不为explicit，允许int-to-Rational隐式转换
    int numerator() const;
    int denominator() const;
    //你想做乘法，于是你写了个成员函数
    const Rational operator* (const Rational& rhs) const;
private:
    ...
};
```
虽然正常运作没有问题，但是混合运算还是不太好：
```c++
result = oneHalf * 2;//ok
result = 2 * oneHalf;//wrong!
result = 2.operator*(oneHalf);//一目了然了。
```
当然，编译器也会尝试寻找可被以下这般调用的`non-member operator*`(在命名空间内)
```c++
result = operator*(2,one);
```
为什么交换过来就会出现问题？
### 隐式类型转换
编译器知道你在传递一个int，而函数需要一个Rational，于是它就真的这么做了。
```c++
const Rational temp(2);
result = oneHalf * temp;
```
所以一旦Rational的构造函数为explicit，上面那个ok也会不ok起来。
结论是：只有当参数被列于参数列时，这个参数才是隐式类型转换的合格参与者。
有点像...Haskell的函数调用法，前面的是第一个参数，后面的是第二个参数。这里是第一个是对象本身（this），后面的是参数。

### non-member函数
所以真正的实现应该是
```c++
class Rational {
    ...
};
const Rational operator*(const Rational& lhs,const Rational& rhs)
{
    return Rational(lhs.numerator() * rhs.numerator(),
                    lhs.denominator() * rhs.denominator());
}
Rational oneFourth(1, 4);
Rational result;
result = oneFourth * 2;//ok
result = 2 * oneFourth;//ok
```
剩下需要关心的就只是关于`operator*`是否应该是friend呢？

然而本法则中的真理在Template C++中并让Rational成为class template时，又有了新的争议解法和惊人的设计牵连，我们以后再谈。

## 25:考虑写出不抛出异常的swap函数
### default swap
swap实现如下：
```c++
template<typename T>
void swap(T& a, T& b){
    T temp(a);//好像实际用的赋值运算符，不过应该各种版本都有
    a = b;
    b = temp;
}
```
### member swap
但是如果你需要一个只交换指针的swap，你需要自己写：
```c++
class WidgetImpl {//针对Widget设计的class
public:           //细节不重要
    ...
private:
    int a,b,c;//may be many
    std::vector<double> v;//意思是复制时间很长
    ...
};
class Widget {
public:
    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs)
    {
        ...
        *pImpl = *(rhs.pImpl);
    }
    ...
private:
    WidgetImpl* pImpl;
};
namespace std {
    template<>
    void swap<Widget>( Widget& a,Widget& b)
    {
        swap(a.pImpl, b.pImpl);
    }
}
```
### non-member swap
很遗憾，这个函数不能通过编译：因为pImpl指针是private。
我们可以将这个特化版本声明为friend，但和平时不太一样：
```c++
class Widget {
public:
    ...
    void swap(Widget& other)
    {
        using std::swap;
        swap(pImpl, other.pImpl);
    }
    ...
};
namespace std {
    template<>
    void swap<Widget> (Widget& a, Widget& b)
    {
        a.swap(b);
    }
}
```
这样就好啦。还和STL容器有一致性，因为stl也是这样做的。
然而如果Widget和WidgetImpl都是class template的话...

### std::swap的特化
```c++
template<typename T>
claass WidgetImpl { ... };
template<typename T>
class Widget { ... };
namespace std {
    template<typename T>
    void swap< Widget<T> > (Widget<T>& a, Widget<T>& b)
    { a.swap(b);}
}
```
突然不合法。c++只允许对类模板偏特化，对函数模板施行偏特化是行不通的。
我查了一下，这个准则现在也适用。
一种偏特化函数模板的做法是添加一个重载版本：
```c++
namespace std {
    template<typename T>
    void swap (Widget<T>& a,Widget<T>& b)//wrong
    { a.swap(b); }
}
一般而言，重载函数模板没有问题，但std是个特殊的命名空间，其管理规则也比较特殊。客户可以全特化std内的templates，但不可以添加新的template进去。

现在我们还是需要一个更高效的swap但template特定版本。这时候我们的答案还是声明一个non-member swap让它调用member swap，只是不再声明为`std::swap`的特化或重载版本。
```c++
namespace WidgetStuff {
    ...
    template<typename T>
    class Widget { ... };
    ...
    template<typename T>
    void swap(Widget<T>& a,Widget<T>& b)//not in std
    { a.swap(b); }
}
```
记得那个`using std::swap`吗？你明白了吗？
注意，如果你不特意说明，编译器会优先寻找T专属的swap，即特化过的std::swap版本。

需要小心这种调用：
```c++
std::swap(obj1,obj2);
```
某些迷茫的程序员的确这样用了，而这就是你的类对`std::swap`进行全特化的重要原因。

### 总结
1. 提供一个public swap成员函数，绝不该抛出异常
2. 在class或template所在命名空间内提供一个non-member swap，令他调用上面的swap
3. 为你编写的class（不是template的）特化std::swap，令他调用你的swap函数
4. 如果你调用swap，确定包含一个using声明。

5. swap一般用以提供极高的异常安全性保障，但是基于一个假设——成员函数版本的swap绝不抛出异常

这段有点长，多读几遍。

# 本期后记
写着写着就冗长了起来，实在是单独的文字说明没有感染力也不会让人印象深刻。好在本书的例子都足够、典型。
那就没什么好办法了，必须经常看才行。笔记只是提供了一个不需要再重新通读，而只需要唤醒记忆的简单途径而已。