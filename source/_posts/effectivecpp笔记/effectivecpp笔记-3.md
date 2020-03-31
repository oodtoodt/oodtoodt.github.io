---
title: effectivecpp笔记-3
date: 2020-02-07 14:07:09
tags:
- old c++
- home

categories:
- c++

---

这篇会有点小长，主要是内容比较多一点。
包括了转型、异常、inline、解耦
<!--more-->

---

# 实现（Implemntations)
本篇介绍了各种在完成实现层面需要注意的各种东西：
+ 过度使用转型导致代码又慢又难维护，还会有微妙的错误
+ 太快定义变量导致效率过低
+ 返回对象的「内部句柄」可能会破坏封装并留下「悬挂句柄」
+ 未考虑异常的冲击导致的资源泄漏和数据破坏
+ 过度inline导致代码膨胀
+ 过度耦合(coupling)导致过长的build时间

## 26:尽可能拖后变量定义的出现时间
+ 异常可能使得你的变量没有被使用，所以拖后定义的出现时间
+ 尽可能的直接初始化，而不要赋值等等无用操作
+ 所以应当尽量延后到变量能够给到实参位置。
+ 对循环来说，考察赋值成本与一组构造+析构成本的对比，决定变量定义于循环内/外。

## 27:少做转型。
>c++的设计目标之一是保证类型错误绝不可能发生
这是一个极具价值的保证：表示如果你通过编译，那它就不企图在任何对象上执行不安全、无意义的操作
不幸的是，转型（cast）破坏了类型系统。这导致出了各种各样的或显著或隐晦麻烦。

### 转型
+ 旧式转型：T(expression)、(T)expression

+ 新式转型
```c++
const_cast<T>{}
//常量性转除
dynamic_cast<T>{}
//安全向下转型
reinterpret_cast<T>{}
//执行低级转型
static_cast<T>{}
//强迫隐式转换
```
题外话：remove_const和const_cast之间的区别联系？
用到旧式转型的场景是explicit构造传递函数
```c++
class Widget{
public:
    explicit Widget(int size);
};
void doSomeWork(const Widget& w);
//---函数风格
doSomeWork(Widget(15));
//---c++风格的，有些蓄意，不怎么像转型
doSomeWork(static_cast<Widget>(15));
```

注意：单一对象可能有一个以上的地址，在继承中几乎一直发生着。你应当避免做出「对象在c++中如何布局」的假设，也不要以此为基础做转型，比如转成`char*`来进行指针算术，基本上都是未定义行为。
这种布局方式和地址计算方式随编译器不同而不同，「由于知道对象如何布局」而设计的转型在某一平台行得通，在其他平台不一定行得通。

### 一个很标准的错误
指我们进行转型的时候会写出..似是而非的代码

```c++
class Window {
public:
    virtual void onResize() { ... }//base onResize实现代码
    ...
};
class SpecialWindow: public Window {
public:
    virtual void onResize() { 
        static_cast<Window>(*this).onResize();
        //来了！
        //将*this转型成window，然后调用它的onResize
        //血崩
        ...//进行SpecialWindow专属行为
    }
}
```
然而它调用的OnResize并不是当前对象上的函数哦，是稍早转型动作说建立的一个「`*this`对象的基类成分」的暂时副本身上的onResize。
就是说你新建了个副本，然后操作一番，和你本体p关系没有。哦倒也不是，你还进行了专属行为。处于一种...扯裂的状态。
你只是想调用基类的函数请这么写：
```c++
virtual void onResize() {
    Window::onResize();
}
```

### dynamic_cast
首先，这玩意相当的慢。深度继承或多重继承的成本更高！有些必须支持动态连接使得必须使用strcmp，要慎重考虑。
通常你用这东西是因为你想在一个你认定为derived class对象身上执行derived class操作函数，但手上只有一个指向基类但pointer或者reference。有两个一般性的做法可以避免这个问题：
+ 使用容器并在其中存储直接指向继承对象的指针（通常是智能指针）如此便消除了通过「基类接口处理对象」的需要。假设先前的Window/SpecialWindow体系中只有SpecialWindows才支持闪烁效果：
```c++
class Window{...};
class SpecialWindow:public Window {
public:
    void blink();
    ...
}
typedef std::vector<std::tr1::shared_ptr<Window> > VPW;
VPW winPtrs;
for(VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) {
    if(SpecialWindow* psw = dynamic_cast<SpecialWindow* > (iter->get()))
        psw->blink();
}
```
这坨看着就难过，using+auto大法好。可惜这时候没有

不应该使用dynamic_cast,if都去掉直接
```c++
{(*iter) -> blink();}
```
当然这样无法在同一容器内存储指针指向所有可能各种的Window派生类。注意让多种类型的多个容器都具备类型安全性。

+ 另一种方法是在基类里提供base class接口处理「所有可能的Window派生类」（通过提供virtual函数）。
注意缺省什么都不要做
绝对要避免所谓的连串dynaminc_cast，即下面的东西：
```c++
class Window { ... };
...//derived class定义
//VPW如上，如果是提供virtual那么智能指针里应该放Windows而不是继承类。
//for如上
{
    if(SpecialWindow1 * psw1 = dynamic_cast<SpcialWindow1*>(iter->get())){...}
    else if(SpecialWindow2 * psw2 = dynamic_cast<SpcialWindow2*>(iter->get())){...}
    else if(SpecialWindow3 * psw3 = dynamic_cast<SpcialWindow3*>(iter->get())){...}
    ...
}
```
这个代码又大又慢，而且每次Window class继承体系有改变，所有此类代码必须检查是否需要更改。
这样的代码应该总是被「基于virtual函数调用」取代。

正如前面所说，如果可以，尽量避免转型。
如果转型是必要的，尝试把它隐藏在函数的背后，而不是让客户将转型放进他们的代码里。

## 28:避免返回「handles」指向对象内部部分
### 例子
比如你想要一个矩形。你选择使用左上角和右下角两个「点」表示。于是你表示了点
而且你需要给客户提供计算Rectangle范围的upperleft和lowerRight函数。
```c++
class Point{
    ...
};
struct RectData{
    Point ulhc;
    Point lrhc;
};
class Rectangle{
public:
    ...
    Point& upperLeft() const{ return pData->ulhc; }
    Point& lowerRight() const{ return pData->lrhc; }
    ...
private:
    std::tr1::shared_ptr<RectData> pData;
};
```
可以通过编译但是却是错误的。
问题出在在你想返回引用来获得更高效率，又不想让客户修改所以放了const，但是两个函数实际返回了指向private内部数据的引用，它是可以改变的。可以直接`rec.upperLeft().setX(50);`

这给我们带来两个教训：
1. 成员变量的封装性最多只等于「返回其引用」的函数的访问级别。这里的private其实是假的，根本上是个public。
2. 如果const成员函数传出一个reference，这个引用所指的数据与对象自身有关联，它又被存储在对象之外，那么这个函数的调用者可以修改那笔数据。

为什么会这样呢？

（个人感觉，这里涉及到两个点，const的伪性和下面会说的句柄问题。const不是说我这里就一定是它不会改变了，见法则3好了）

因为引用、指针、迭代器统统都是所谓的handles（句柄），而返回一个「代表对象内部数据」的句柄意味着降低对象的封装性和const被更改。
**注意对象内部数据不只包含变量，还有不公开的那些函数，不要返回他们的句柄。**

我们遇到的const问题只需要把返回值也变成const就好了，意味着不能更改返回值，也不能在函数里更改它：
```c++
const Point& upperLeft() const { ... }
```
但这还是会导致另一个致命的问题：悬挂句柄——所指的东西不复存在，比如说：
```c++
class GUIObject { ... };
const Rectangle boundingBox(const GUIObject& obj);
...
GUIObject* pgo;
...
const Point* pUpperLeft = &(boundingBox(*pgo).upperLeft());
```
获得一个新的、暂时的Rectangle对象。我们且称之为temp。随后upperLeft()作用在temp上，返回一个引用，pUpperLeft指向这个Point...行了大家都看懂了，这个语句结束了pUpperLeft立刻就变成悬挂句柄。

但是有时候你必须用，比如`operator[]`，尽量少用，尤其是指向对象内部数据的时候

## 29:力求「异常安全」的代码
>异常安全性(Exception safety)有几分像是...呃...怀孕。但等等，在我们完成求偶之前，实在无法确实的谈论生育。
让我们看一个糟糕的异常安全的例子，我们需要有个class用来表现夹带背景图案的GUI菜单，用于多线程，使用mutex（互斥器）作为并发控制用。
```c++
class PrettyMenu {
public:
    ...
    void changeBackground(std::istream& imgSrc);
    ...
private:
    Mutex mutex;
    Image* bgImage;
    int imageChanges;
};
//一个可能的实现，糟糕！
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    lock(&mutex);
    delete bgImage;
    ++imageChanges;
    bgImage = new Image(imgSrc);
    unlock(&mutex);
}
```
异常安全有两个条件，而这个函数没有满足任何一个条件：
+ 不泄漏任何资源
一旦new Image导致异常，对unlock的调用就绝不会执行。
+ 不允许数据败坏
一旦new Image抛出异常，bgImage就永远指向了一个已被删除的对象，imageChange也以累加。

解决资源泄漏很容易，在13和14中我们讨论过用对象管理资源，导入Lock class作为一种「确保互斥器被及时释放」的方法:
```c++
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    Lock ml(&mutex);
    delete bgImage;
    ++imageChanges;
    bgImage = new Image(imgSrc);
}
```
好了现在我们可以专注于解决数据败坏问题了。不过我们先要面对一些术语

### 异常安全函数 “规范”
异常安全函数提供以下三个保证之一：
+ 基本承诺：如果异常被抛出，程序内的任何事物仍然在有效状态下。没有任何对象或数据结构会因此而败坏，所有对对象都处于一种前后一致的状态。然而程序的现实状态恐怕不可预料。
+ 强烈保证：如果异常被抛出，程序状态不改变。调用这样的函数需要有这样的认知：如果函数成功，就是成功，失败则会回到调用之前的状态
这种强烈保证比上条更加严格，程序只有两种可能的状态了。
+ 不抛出保证：保证绝不抛出异常。作用于内置类型（int、指针等等）身上的所有操作都提供了nothrow保证。


异常安全代码必须提供上述三种保证之一。因此我们只需要角色我们所写的每一个函数提供哪一种保证？除了一些不具异常安全性的传统代码，你应该只在一种情况下不提供异常安全保证：...你的天才团队确认你的程序有泄漏并败坏数据的需要。

nothrow是最想被构建的，但是基本上无法实现。

对于我们上述的例子而言，提供强烈保证不困难。
首先改变ngImage的类型为智能指针以方便我们防止资源泄漏。
第二，我们重新排列changeBackground内的语句顺序，使得在更换图像之后才累加imageChanges。
```c++
class PrettyMenu {
    ...
    std::tr1::shared_ptr<Image> bgImage;
    ...
};
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    Lock ml(&mutex);
    bgImage.reset(new Image(imgSrc));
    ++imageChanges;
}
```
不需要delete，不需要unlock。
美中不足的是参数imgSrc，如果Image构造函数抛出异常，有可能输入流的读取记号被移走，这是一种可见状态改变，所以解决这里之前都只是基本的保证。
但我们暂且不提（认为你有能力想出办法过度，比如改变参数类型、比如从istream改为一个内涵图像数据但文件名称等），先来说一个很典型的一般化的设计策略：

### copy and swap
它会导致强烈保证，原则很简单：
为你打算修改的对象做出一份副本，在副本上做出所有必要修改，若有任何修改动作抛出异常，原对象仍保持未改变状态。改变都做完了在一个不抛出异常的操作中进行swap。
实现上通常被称为pimpl idiom
对PrettyMenu而言，典型写法如下：
```c++
struct PMImpl {
    std::tr1::shared_ptr<Image> bgImage;
    int imageChange;
};
class PrettyMenu {
    ...
private:
    Mutex mutex;
    std::tr1::shared_ptr<PMImpl> pImpl;
};
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    using std::swap;
    Lock ml(&mutex);
    std::tr1::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl));
    pNew->bgImage.reset(new Image(imgsrc));
    ++pNew->imageChanges;
    swap(pImpl,pNew);
}
```
struct的原因是因为PrettyMenu的数据封装性已经有雨pImpl是private而获得了保证。这里更多的考虑打包和方便的问题。

copy-and-swap策略是对对象做出全有或者全无的很好办法，但不能保证整个函数有强烈异常安全性，处于原因，我们考虑下面的调用：
```c++
void someFunc()
{
    ...//local 状态做一份副本
    f1();
    f2();
    ...//修改后的状态置换过来
}
```
显然，f1和f2的异常安全性影响到了someFunc的安全性。然而，即使f1f2皆强烈安全，情况也无有好转：如果f1圆满结束，程序状态可能改变，而f2随之异常，程序状态前后并不相同。
这里就设计到另一个问题：

### 连带影响&效率
如果函数只操作局部性状态，强烈保证就比较容易。函数对非局部性数据有连带影响时，提供强烈保证就困难得多。举个例子（上面那里），如果调用f1带来的影响是某个数据库改动了，那就很难让someFunc具备强烈安全性。

另一个绝对严重的问题是效率。copy本身这个操作非常让人头疼。如果那空间和时间是我们无法承担的，那我们也无法做到强烈保证。不知道对于c++11的各种移动函数而言，这里有没有更好的解决。就我个人感觉，不能。就算副本是即刻销毁右值也无济于事，想要异常安全的这个过程使得它必须复制操作并移动，优化空间看起来非常小。

这时候你只能退而求其次，选择基本保证。
记住，追求强烈保证很好，但如果那效率和复杂度使人难以承受或不可行，选择基本保证是没有问题的。

### 关于怀孕
一位女性若非怀孕，就是没怀孕。不可能说她「部分怀孕」。
如果一个软件不具备异常安全性，那就全盘否定，没有所谓局部异常安全的说法。

## 30:了解inlining的里里外外。
你可能已经知道：inline的整体理念是把函数以本体替换。所以关键在于对短小的函数进行inline达到比函数调用更小的码从而达到更高的指令高速缓存击中率。

+ inline只是对编译器的一个申请，不是强制指令。
隐喻的inline是在类中定义函数，这样的函数都是inline的（包括friend函数）

+ 编译器通常不对「函数指针」调用的函数实施inline。

+ 类中定义的函数是inline

+ 构造和析构通常不应该为inline。
继承里面代码会膨胀爆炸。即使没有继承，本身初始化类中变量也其实是有很多默认的代码的。

+ inline函数通常一定被置于头文件内
因为大多数build环境在编译过程中进行inline，所以编译器需要知道那个函数长什么样子。
关键字 inline 必须与函数定义体放在一起才能使函数成为内联，仅将 inline 放在函数声明前面不起任何作用。

+ inline的特性与编译器息息相关
有的函数可能可以inline。
有的函数可能inline的时候会生成一个函数本体。

即是说，一个inline函数是否真的inline，主要取决于编译器。

+ inline无法随着程序库的升级而升级

+ inline难以调试（调试器不认），试着在inline里设断点？

+ 尝试优化80-20经验法则：平均80%的执行时间在20%的代码里。这是一个重要的法则，你也应当为这20%进行inline或者瘦身。但必须目标准确。

### 关于inline的题外话
下面就不是effectivecpp里的内容了。
关于inline的几点：
>1. 不要再把 inline 和编译器优化挂上关系了，太误导人。编译器不傻，inline is barely a request。你不加inline，小函数在开O3时，编译器也会自动给你优化了。看到inline时，应该首先想到其他用意，在考虑编译器优化。

>2. inline最大的用处是：非template 函数，成员或非成员，把定义放在头文件中，定义前不加inline ，如果头文件被多个translation unit（cpp文件）引用，ODR会报错multiple definition。

>3. 谨慎使用 static：如果只是想把函数定义写在头文件中，用 inline，不要用static。static 和 inline 不一样：
static 的函数是 internal linkage。不同编译单元可以有同名的static 函数，但该函数只对 对应的编译单元 可见。如果同一定义的 static 函数，被不同编译单元调用，每个编译单元有自己单独的一份拷贝，且此拷贝只对 对应的编译单元 可见。
inline 的函数是 external linkage，如果被不同编译单元调用，每个编译单元引用／链接的是同一函数，同一定义。
上面的不同直接导致：如果函数内有 static 变量，对inline 函数，此变量对不同编译单元是共享的（Meyer's Singleton）；对于static 函数，此变量不是共享的。看后面的代码就明白区别了。

>4. static inline 函数，跟 static 函数单独没有差别，所以没有意义，只会混淆视听。

>5. inline 函数的定义不一定要跟声明放在一个头文件里面：定义可以放在一个单独的头文件 .hxx 中，里面需要给函数定义前加上 inline 关键字，原因看下面第2点；然后声明放在另一个头文件.hh中，此文件include上一个.hxx。这种用法在boost里很常见：优点1：实现跟API分离，encapsulation（封装）。优点2：可以解决有关inline函数的循环调用问题：这个不展开说了，看一个这个文章就懂了：Headers and Includes: Why and How 第 7 章，function inlining。



我们来看一个例子：
```c++
// 文件A.h 代码如下:

class A
{
public:
    A(int a, int b) : a(a),b(b){}
    int max();
private:
    int a;
    int b;
};
// 文件A.cpp 代码如下:

#include "A.h"
inline int A::max()
{
    return a > b ? a : b;
}

// 文件Main.cpp 代码如下:

#include <iostream>
#include "A.h"
using namespace std;
inline int A::max()
{
    return a > b ? a : b;
}

int main()
{
    A a(3, 5);
    cout << a.max() << endl;
    return 0;
}
```
一切正常编译，输出结果：5

倘若你在Main.cpp中没有定义max内联函数，那么会出现链接错误：
```
error LNK2001: unresolved external symbol "public: int __thiscall A::max(void)" (?max@A@@QAEHXZ)main.obj
```
找不到函数的定义，所以内联函数可以在程序中定义不止一次，只要 inline 函数的定义在某个源文件中只出现一次，而且在所有源文件中，其定义必须是完全相同的就可以。

在头文件中加入或修改 inline 函数时，使用了该头文件的所有源文件都必须重新编译。

新的例子：
```c++
//a.hh
#ifndef A_HH
#define A_HH
#include <iostream>
 
namespace static_test
{
  static int& static_value() // (!*!) Or change this to inline
  {
    static int value = -1;
    return value;
  }
 
  namespace A
  {
    void set_value(int val);
    void print_value();
  }
}
#endif

//a.cc
#include "a.hh"
 
namespace static_test
{
  namespace A
  {
    void set_value(int val)
    {
      auto& value = static_value();
      value = val;
    }
   
    void print_value()
    {
      std::cout << static_value() << '\n';
    }
  }
}

//b.hh:
#ifndef B_HH
#define B_HH
 
#include <iostream>
 
namespace static_test
{
  namespace B
  {
    void set_value(int val);
    void print_value();
  };
}
#endif

//b.cc:
#include "a.hh"
#include "b.hh"
 
int main()
{
  static_test::A::set_value(42);
 
  static_test::A::print_value();
  static_test::B::print_value();
 
  static_test::B::set_value(37);
   
  static_test::A::print_value();
  static_test::B::print_value();
 
  return 0;
}
```
+ a.hh 中标注 `(!*!) `的那行，如果是inline，输出：42，42，37，37。value 在整个程序中是个Singleton。
+ 如果是 static，输出：42，－1，42，37。value 在不同编译单元是不同的拷贝，即使它被标注 static

## 31：将文件间编译依存关系降到最低
这里讨论的是关于你修改了一个class的private成分，然后整个世界都被重新编译和连接了。
c++没有把「接口从实现中分离」这件事做的很好。

+ 标准库不太可能成为编译瓶颈
不过避免使用不受欢迎的头文件

+ 编译器必须在编译器期间知道对象的大小
这样的问题在java、smalltake等语言上并不存在，他们定义对象只分配一个空间给指针用，所以说c++也可以自己尝试这样做——将对象实现细目隐藏在指针背后：
```c++
#include<string> //注意此标准库的组件不该被前置声明
#include<memory> //tr1::shared_ptr
class PersonImpl;
class Date;
class Address;
class Person{
public:
    Person(const std::string& name,const Date& birthday,const Address& addr);
    std::string name() const;//没啥，就正常的一些函数，比如..
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::tr1::shared_ptr<PersonImpl> pImpl;
}
```
这样Person的客户就完全与Dates，Address以及Person的实现分离来，这些class的实现不需要Person重新编译，这就是「接口与实现分离」。
顺带一体，这种类中只有一个指针成员指向其实现类的叫「pimpl idiom」，这种指针一般就叫pImpl。（pointer to Implementation）
而像Person这种用pimpl idiom的类，也被叫Handle class（句柄类）。

### 分离
分离的关键在于以「声明的依存性」替换「定义的依存性」，这就是编译依存性最小化的本质。所以我们应尽量做到：
+ 如果使用对象引用和对象指针可以完成任务，就不要使用对象本身。
在于需要用到该类型的定义式。说白了还是拷贝那一套。
+ 如果能够，以类的声明式替换定义式。
```c++
class Date;//声明式
Date today();
```
+ 为声明式和定义式提供不同的头文件。
```c++
#include "datefwd.h"//声明，但未定义Date。
Date today();
void clearAppointments(Date d);
```
这让人想到那些我见过的全是声明但是没有定义的头文件。
比如`<iosfwd>`，包括iostream各组件的声明式，而且说明这个法则适合template：
```c++
template <class _CharT, class _Traits = char_traits<_CharT> >
    class _LIBCPP_TEMPLATE_VIS basic_iostream;
...
typedef basic_streambuf<char>        streambuf;
typedef basic_istream<char>          istream;
typedef basic_ostream<char>          ostream;
typedef basic_iostream<char>         iostream;
```

---

句柄类想做事情的方法有两种，一种是把所有函数转交给相应的实现类。
下面是关于Person的实现：
```c++
#include "Person.h"//Person class 定义式
#include "PersonImpl.h"//PersonImpl的class定义式，注意和上面那行具有完全相同的成员函数，两者接口完全相同
Person::Person(const std::string& name,const Date& birthday,const Address& addr)
:pImpl(new PersonImpl（name, birthday, addr)）
{}
std::string Person::name() const
{
    return pImpl->name();
}
```
另一种是把Person变成抽象基类（abstract base class），称为Interface lass。这种类的目的是一一描述继承类的接口，通常无成员变量，无构造函数，只有一组纯虚函数和虚析构函数来叙述整个接口。
这种就比较类似Java和.NET的Interface，但c++没这个概念，也不需要负担相应的责任（比如c++可以添加成员变量和成员函数），也就有了更大的弹性。
实现而言就是
```c++
virtual std::string name() const = 0
```
Interface class创建新对象一般调用一个特殊函数，称为factory函数或virtual构造函数。他们返回指针或智能指针，指向动态分配所得对象，而该对象支持Interface class的接口，这样的函数往往在接口类内被声明为static。
```c++
class Person{
public:
    static std::str1::shared_ptr<Person> create(const std::string& name,...);
    ...
};
//---
std::string anme;
...
std::tr1::shared_ptr<Person> pp(Person::create(name,...));
```

### 代价
对Handle class而言，成员函数必须通过implementation pointer取得对象数据。那会为每一次访问增加一层间接性。同理内存也会增加。最后，你还将遭受实现类（Implementation）动态内存分配。
对于Interface class，每隔函数都是virtual，所以每个调用都带着间接跳跃的成本，还带着vptr（虚函数那套代价）。
最后，不论Handle class还是Interface class，一旦脱离inline函数都没什么太大作为。条款30解释了函数本体为了被inline必须（很典型的）置于头文件内（因为是编译期行为），但这两者正式特别被设计来隐藏实现细节的。
当这些代价导致的冲击过大以致于类之间的耦合已经不是关键的时候，就用具象（体）类（concrete class）替换这两者吧。

### 总结
支持编译依存性最小化的一般构想是：相依于声明式，不要相依于定义式。
基于此构想的两个手段：Handle class，Interface class。
程序库头文件应该以「完全且仅有声明式」的形式存在。这点也适用于template
