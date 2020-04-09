---
title: effectivecpp笔记-1
date: 2020-01-23 23:17:54
tags:
- old c++
- home

categories:
- c++
- book_notes

---

这书确实好,这里是第一、二章，从法则01至12
<!--more-->

---

# 前言
一如传言中的那样，有无读过此书的c++程序员是两个层次。
确实，这本书讲到了那些c++里隐蔽的、必要的许多知识。而且充满了实用性。
唯一的缺憾就是这本书实在是太早了（06年），那个时候c++11c++14c++17还没出世，c++还处在一个...混沌的开化时期（但是各个系统，尤其是模板、stl都已经几乎成型极具威力了），很多东西现在可以很方便的就避开，而无需像书中那样曲折的去实现——当然这种对照可以使我们更清晰。

如果篇幅不长，我们不设3级标题，而仅仅以无序列表形式提供内容。


# 让自己习惯c++
## 01:视c++为语言联邦
+ c-part
+ ◊object-oriented c++(oop里的oo)
+ Template c++
+ stl

## 02:尽量以const、enum、inline替换#define
inline是宏的范畴，前二者用于常量。
现在有了新的constexpr了
好像还有啥，记不得了
using好像不太符合这几个的内容（指#define优化）

## 03:尽可能的使用const
尤其是禁止如(a * b) = c这种操作：返回const
然而提到了const的各种问题：
+ 常量性不同可以被重载(这不是问题，是要注意的点)
+ bitwise constness（无一可变）与logical constness（一定范围可变，客户端侦测不出）——引出了mutable，可以在const中修改的变量
注意很多成员函数不具备const性质却能通过bitwise测试，比如const函数返回reference指向对象内部。
mutable我第一次还是在lambda里见到的...可以对照。
+ 令non-const调用const避免大量重复内容
以下是一个关于const与non-const的例子。
```c++
class TextBlock{
public:
    const char& operator[](std::size_t position) const
    {
        ...
        return text[position];
    }
    char& operator[](std::size_t position){
        return
            const_cast<char&>{
                static_cast<const TextBlock&>(*this)
                    [position]
            };
    }
};
```
但是
这个两次转型(casting)总是看着非常别扭，不知道后面有没有更优雅的实现？

这里提一嘴关于const和volatile
“const”含义是“请做为常量使用”，而并非“放心吧，那肯定是个常量”。

“volatile”的含义是“请不要做没谱的优化，这个值可能变掉的”，而并非“你可以修改这个值”。

## 04:确定对象使用前已被初始化
### 总是初始化。
尤其是在c++11统一了初始化规范（初始化列表）之后，有了标准，但是习惯不要落下。
而且c++11有了类成员的内部初始化。
   
>要记住这么多种初始化规则，并从中选用最合适的一种，绝非易事。
>C++11的解决方法是对于所有的初始化，均可使用“{}-初始化变量列表”

### 按变量声明次序初始化
而且按照先base后derived（继承）的顺序

### 建议用引用函数来初始化非本地静态变量
(singleton的思想)
最常见为多编译单元内non-local static对象由「模板隐式具现化」形成
这里的static指的是作用域global或位于namespace内，或者在class或filne内被声明为static，简单的例子是
extern变量1，变量2使用变量1的某个函数，此时由于这两个变量是「不同的人在不同的时间在不同的源码文件建立起来的，是定义于不同编译单元内的non-local static对象」，无法确定变量1会在变量2之前被初始化。

就是说，在调用之前我其实是用了个引用函数建立了static对象，在这个函数里我其实必然会进行初始化，所以就避免了所有的次序问题。且如果不调用就无构造析构成本。且是绝佳inline优化对象。
但是内涵static对象使他们在多线程中具有不确定性。一种做法是在单线程启动阶段全部调用一遍

这一点不知道在新的版本中有无对应措施，有待继续学习。

# 构造/析构/赋值 运算
constructors,Destructors,and Assignment Operators
## 05:了解c++默默编写并调用了哪些函数
指默认构造、拷贝构造、析构函数、拷贝赋值运算符
以及新的移动构造函数、移动赋值运算符

但是默认的函数也是...有操作的。

### 问题
这里有一个问题：为什么默认的拷贝构造函数可以识别并调用已有的拷贝构造函数？
```c++
template<typename T>
class NameObject{
public:
    NameObject(const char* name, const T& value);
    NameObject(const std::string& name, const T& value);
    ...
private:
    std::string nameValue;
    T objectValue;
};
NameObject<int> no1("Smallset Prime Number",2);
NameObject<int> no2(no1);
```
这里no2默认的拷贝构造函数会调用string的拷贝构造函数以no1.nameValue来对no2的string初始化，int内置类型直接拷贝每bit。

### 标准
我也不推测了，直接上标准：（复制构造函数）

>初始化：`T a = b; `或 `T a(b);`，其中 b 类型为 T；
>函数实参传递：`f(a)`;，其中 a 类型为 T 而 f 为` Ret f(T t)`；
>函数返回：在如 `T f() `这样的函数内部的` return a;`，其中 a 类型为 T，它没有移动构造函数。

---

>若隐式声明的复制构造函数未被弃置，则当其被 ODR 式使用时，它为编译器所定义（即生成并编译函数体）。
>对于 union 类型，隐式定义的复制构造函数（如同以 std::memmove）复制其对象表示。
>对于非联合类类型（class 与 struct），**该构造函数用直接初始化，按照初始化顺序，对对象的各基类和非静态成员进行完整的逐成员复制。**
>若它满足对于 constexpr 构造函数的要求，则生成的复制构造函数为 constexpr。 (C++11 起)
>当 T 拥有用户定义的析构函数或用户定义的复制赋值运算符时，隐式定义的复制构造函数的生成被弃用。(C++11 起)

注意到加粗部分：用直接初始化进行完整对逐成员复制。复制=拷贝（copy）。直接初始化，其实就是`T a = "aaa"`这种形式，就是另一个拷贝构造函数了。
可以理解为优先调用拷贝构造函数，再考虑内置类型逐位复制。

### 更多牵涉...
这里其实还牵涉到了关于平凡(trivial)，也就是POD定义的问题，以及复制消除(copy-elision)的问题，还是很复杂的。

写到这里我意识到我迟早要搞一篇专门弄一下这几个十分基础却很...难搞的东西。
c++无易事啊。

### 我们继续...
其实已经偏题蛮多了。+++。
默认的拷贝赋值运算符产生条件比较苛刻。
注意到c++11中到default，所以这里理解还是很重要的。

## 06:如果不想要编译器产生的函数，就要明确拒绝。
比如说阻止拷贝。于是对于拷贝构造函数或者拷贝赋值运算符...声不声明都很难受。
+ 使用private声明拷贝构造函数与拷贝赋值运算符。
+ 使用一层private继承来实现。

不过针对这一问题，其实已经在c++11有了解决实现：delete。

## 07:为多态基类声明virtual析构函数
对于factory(工厂)函数，即一「返回一个base class指针，指向新生成的derived class对象」的函数，会在delete的时候出现问题：
一指针指向 继承类 的对象，而那个对象却经由一个 基类 的指针被删除。
c++明确指出这是一个未定义行为。
解决方法：基类声明virtual析构函数。

### 任何带有virtual函数几乎确定应该有一个virtual析构函数。
（原因是这样的类几乎都是基类）


### 如果无virtual，一般意味不做基类，此时声明virtual函数是馊主意：
virtual函数会为对象增加一份vptr指针和由vptr指向的数组vtbl(虚函数指针和虚函数表)，维护这些东西会使其失去移植性（对象变大）。（除非你明确补偿vptr）

### 一个例子
 标准string不带任何virtual函数，但有时程序员会错误的将其作为基类：
```c++
class SpecialString: public std::string{
    ...
};
```
此时如果你在程序中无意将一个pointer-to-SpecialString转换为一个pointer-to-string，然后delete掉那个string指针，你立刻就会被流放到行为不明确的恶地上。
```c++
SpecialString* pss = new SpecialString("Impending Doom");
std::string* ps;
ps = pss;
delete ps;
```
（老实说我写这个例子还用了好长时间，我确实不太熟悉这些东西。这里是资源泄漏，不会立刻boom。）

### 禁止派生
>「很不幸c++无有一个final classes或sealed classes那样的禁止派生机制」

很不幸我们在c++11里就有了final关键字。

### pure virtual destructor
为构建抽象对象可以特意使出纯虚析构函数。但是注意，你必须为这个函数提供一份定义。

#### 析构函数的运作
最深层继承的class先调用析构函数，逐层调用基类。编译器会在纯虚类的派生类的析构函数中创建一个对纯虚类析构函数的调用动作。

### 并非所有基类设计为多态用途
标准string和stl都不被设计为作为基类使用。
有些作为基类，但是不是为了多态用途。这样的classes如
+ Uncopyable
+ input_iterator_tag
等等

## 08:别让异常逃离析构函数
c++并不禁止析构函数吐出异常，但不鼓励你这么做。
主要原因是析构可能会大量调用，这些异常太多了。两个异常同时存在不是结束程序就是导致不明确行为。
为了解决这个问题，你可以
+ 如果抛出异常就结束程序。
+ 吞下发生的异常。
一般而言，吞掉是坏主意，因为它压制了“某些动作失败”的重要信息。但优势也比负担草率结束程序或不明确行为带来的风险好。

一个较佳的策略是重新设计接口，把调用某些析构函数中需要调用、且可能抛出异常的函数的责任从析构函数交给类客户（别的程序员、使用者）手上。
```c++
class DBConn{
public:
    ...
    void close()
    {
        db.close{};
        closed = true;
    }
    ~DBConn()
    {
        if(!closed){
            try{
                db.close();
            }
            catch(...){
                制作运作记录，记录close调用失败。
                ...
            }
        }
    }
private:
    DBConnection db;
    bool closed;
};
```

## 09:绝不在构造和析构过程中调用virtual函数
所有事情的起因是继承类对象内的基类成分会先行构造。而如果这个构造函数中调用虚函数(即在继承类中重载过的...嗯考虑到不熟悉我就提一嘴)，而如果你调用继承类的构造函数，这个虚构函数是基类的成分！
我们去掉这些曲里拐弯的成分，来看这里的核心问题：
在base class构造函数virtual函数绝不会下降到derived classes阶层。
理由很简单，继承类的成员变量此时未被初始化，则此虚函数如果调用这些变量就会「变成一张通往不明确行为和彻夜调试大会的直达车票」
但有更根本的原因，在派生类的基类构造期间，对象类型是基类。当然如果你究编译器理由，他还是会告诉你上面那条。

析构函数同理。

### 多个构造函数时
每个都需要执行某些相同工作，那么一种做法是把共同的初始化代码放到一个初始化函数如init内。
但这样做通常会隐藏编译器和连接器原本会报出的问题。
比如在init中调用pure virtual函数会终止程序，但如果这个函数不是pure而只是virtual的此时建立派生类的对象会调用基类版本的此函数。
一种解决方法是修改结构，总之不要有virtual在构造和析构中。

当然现在允许了成员定义时初始化，可以省去一些多构造函数的问题。

## 令operator=返回一个reference to *this
考虑如下的解析
```c++
int x,y,z;
x = y = z = 15;// <==>
x = (y = (z = 15));
```
为了实现这种连锁赋值，赋值操作符必须返回一个指向操作符左侧的实参的引用。适用于所有的赋值运算(`+=,*=`等等)

注意这只是协议，并无强制性。

## 在operator= 中处理「自我赋值」
```c++
class Widget{...};
Widget w;
w = w;
//潜在的自我赋值
a[i] = a[j];
*px = *py;//如果恰巧指向同一个东西
```
这些并不明显的自我赋值，是「别名」带来的结果。这里所谓「别名」指一个以上的方法指称某对象。
以下是问题
```c++
class Bitmap{...};
class Widget{
    ...
private:
    Bitmap* pb;
};
Widget& Widget::operator=(const Widget& rhs)
{
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```
如果此时rhs和`*this`是同一个对象，那么就会先删掉自己的东西，然后在赋值...嗯，最后指向一个已经被删除的对象。

### 解决法：调整语序
传统做法是藉由一个证同测试来检验（就看我是不是我），但这仍存在异常方面的麻烦。
我们后面讨论异常安全性，这里注意到「精心安排的语句即可导出异常安全（以及顺带的，自我赋值安全）的代码」就够了。
```c++
Widget& Widget::operator=(const Widget& rhs)
{
    Bitmap* pOrig = pb;记住原先的pb
    pb = new Bitmap(*rhs.pb);//令pb指向新的(可能是同一个）*pb的一个副本
    delete pOrig;
    return *this;
}
```

当然这里还有一个关乎效率的问题，要不要放证同测试在你。

### 解决法：copy and swap
```c++
class Widget{
    ...
    void swap(Widget& rhs);//交换*this和rhs的数据。
    ...
};
Widget& Widget::operator=(const Widget& rhs)
{
    Widget temp(rhs);
    swap(temp);
    return *this;
}
```
我总觉得这种东西看着很别扭。不过有了移动构造函数，我们这里的操作可以变得...更高效，但是实现方式大概不会有什么变化了。

如果有看到更好的方式，记得记一下。


## 12:复制对象的全部成分
简单的说就是深拷贝
继承时是最容易忽略的：没有复制继承来的成员变量。

c++11里统一初始化（成员变量初值）（划掉）。其实是using的使用延展有了继承构造函数解决了这里的问题。这里的问题其实可以解决很多。

在之前，你需要：
+ 复制所有local成员变量
+ 调用所有base classes内的适当copying函数

你不该令拷贝赋值操作符调用拷贝构造函数。
你不该令拷贝构造函数调用构造赋值操作符。



