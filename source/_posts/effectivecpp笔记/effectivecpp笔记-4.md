---
title: effectivecpp笔记-4
date: 2020-02-17 01:56:48
tags:
- old c++
- home

categories:
- c++

---

这篇会有点小难
包括了
<!--more-->

---

# 继承与面向对象设计
Inheritance and Object-Oriented Design
c++的OOP（面向对象编程）和其他的...稍有不同：
public,protected,private
virtual,non-virtual——继承连接
virtual,non-virtual,pure virtual——成员函数
缺省参数和virtual之间的迷惑行为
继承影响c++的名称查找规则

## 32:确定你的public继承塑模出is-a关系
is-a...这个说法就很蠢。
意思就是说，如果class D以public继承class B（Derived，Base），便是说每一个D也是一个B，但反之不成立。
典型的例子就是，学生——人的关系

但是直觉有时候又会出现问题：
企鹅是一种鸟，但企鹅会飞吗？

那么我们来到了这样的一种继承：
```c++
class Bird{
    ...//没有声明fly函数
};
class FlyingBird: public Bird{
public:
    virtual void fly();
    ...
};
class Penguin: public Bird{
    ...//没有声明fly函数
};
```
注意，有时候不去区分会飞的鸟和不会飞的鸟会更好。
还有一种思路，所有鸟都会飞，但是企鹅不会：
```c++
void error(const std::string& msg);//定义于别处
class Penguin: public Bird {
public:
    virtual void fly() { error("Attempt to make a penguin fly!"); }
    ...
};
```

但是这是有差别的，这就是在编译期检测出还是运行期检测出来的区别。好的接口应该防止无效的代码通过编译（见法则18）

然而还是有一些令人困惑的事情：
正方形应该public继承矩形吗？

你的直觉一定是的，但是如果你考虑到一个只增加宽不增加高的函数在正方形上是如何发生的，你就意识到你不可能把正方形当成矩形用，欢迎来到「public继承」的精彩世界。

「是一个（is-a）」，适用于基类上的每一件事情也一定适用于继承类身上。
同样的关系还有两种，我们接下来会认识他们。

## 33:避免遮掩（隐藏）继承名称
核心内容：所谓继承名称其实是作用域的一种外显。即如果画韦恩图，继承类就是在基类里面的一个小圆

所以整个名称查找就是当前作用域找，然后依次递增向上扩散（口胡）。一种类似lua的名称查找（可能根本就是同源的吧，不过让我想起那些hook、反射之类的，就upvalue这种概念，实在很有感觉）

因为如果内部找到了就不会继续向外了，所以又被称作是遮掩、隐藏（hide）

但是没有那么简单，我们还有virtual可以搞事情。

```c++
class Base{
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
};
class Derived: public Base {
public:
    virtual void mf1();
    void mf3();
    void mf4();
};
```
如果我们把遮掩换一种说法，那么mf1和mf3不再被继承...
解决的方法是在继承类中使用using声明式。
还有一种技术叫做转交技术，它是这样的：
```c++
class Derived: private Base{
public:
    virtual void mf1(){//转交函数
        Base::mf1();//inline
    }
}
```
inline转交函数可以实现只继承一个版本的函数，但是注意是在private中的一种用途。

## 34:区分接口继承和实现继承
说的很花里胡哨，实际上我们要区分虚函数、纯虚函数和非虚函数。
### pure virtual
声明pure virtual函数的目的是为了让derived class只继承接口。
但我们意外的可以给纯虚函数提供定义，但调用的唯一方式是明确指出其class名称。

### impure virtual
声明impure virtual函数但目的是让继承类继承该接口和缺省实现。
会提供实现，继承类可能override它。

纯虚函数定义在某些需要对impure virtual函数提供更安全的缺省实现，在于有时你想「提供缺省实现，但除非它们明白要求」，即切断virtual函数接口和缺省实现之间的连接。
```c++
class Airplane {
public:
    virtual void fly(const Airport& destination) = 0;
};
void Airplane::fly(const Airport& destination)
{
    //缺省行为
}
class ModelA:public Airplane{
public:
    virtual void fly(const Airport& destination)
    {
        Airplane::fly(destination);
    }
    ...
};
```

### non-virtual
声明non-virtual函数的目的是为了令dervied class继承函数的接口及一份强制性实现。
意味着它的不变性凌驾于特异性，不论继承类如何特异化，它的行为都不会改变。


### 注意事项
+ 把所有函数都声明为non-virtual的做法很不理想，尤其是non-virtual析构函数会搞砸的（法则7）。当然，一个不会成为基类的class这样是合理的。
不要过分担心virtual函数的效率成本，80-20法则更加重要一些
+ 所有函数都声明是virtual也很不理想。
如果你的不变性的确凌驾于特异性，那就要表达啊出来。

## 35：考虑virtual函数以外的选择
这篇可能啃起来会稍微有些受苦，加油、

为了帮助你跳脱面向对象设计路上的常轨，我们来考虑一下virtual函数之外的一些解法。
比如我现在有一个成员函数healthValue返回游戏中的健康程度，不同人物可能有不同的方式计算它。virtual是直觉的一种思路，现在来看看别的。

### 藉由non-virtual Interface（NVI,非虚接口）手法实现Template Method模式
注意这里的Template跟模板没半毛钱关系。
```c++
class GameChaaracter {
public:
    int healthValue() const
    {
        ...                          //做一些事前工作
        int retVal = doHealthValue();//做真正的工作
        ...                          //做一些事后工作
        return retVal;
    }
    ...
private:
    virtual int doHealthValue() const//继承类可以重新定义它
    {
        ...//缺省
    }
}
```

优点在可以做事前和事后工作，比如互斥器啊、验证函数条件啊等等
换句话说，NVI允许继承类重新定义virtual函数，从而赋予它们控制「如何实现」的权利，但基类保留诉说「函数何时被调用」的权利。
而且没有必要一定得是private，有时候需要是protected，有些时候必须是public，那就不适合NVI了。

### 藉由Function Pointers实现Strategy模式
这个手法主张「人物健康指数对计算与人物类型无关」，不需要「人物」这个成分。
比如可以用构造函数来接受指针指向一个健康计算函数
不过在这之前我要先讲一下typedef免得你看不懂代码

#### typedef
typedef一般是赋别名的，但其实有很多门道
+ 与struct一起用
+ 定义与平台无关的类型
+ 定义别名
这里我们说两点，主要是`char *`和const的问题
```c++
typedef char* PCHAR;
const char* a,b;//后者无定义
char* aa,bb;//后者是char
PCHAR aaa,bbb;//这两个都是char*啦
const PCHAR c,d;//这两个都是char* const，因为...整体考虑并且后置const就是这样的
```
+ 为复杂的声明定义一个新的简单的别名
这太难顶了。
其实是这些复杂声明太难顶了，typedef的过程就像解谜的过程一样。
1. 原声明：
```c++
int *(*a[5])(int, char*);
//指向5个返回类型为int型函数组成的数组的指针变量，变量名为a.
//(*)(int,char*)表示指向一个函数的指针
```
变量名为a，直接用一个新别名pFun替换a就可以了：
```c++
typedef int *(*pFun)(int, char*);
```
原声明的最简化版：
```c++
pFun a[5];
```
2. 原声明：
```c++
void (*b[10]) (void (*)());
```
变量名为b，先替换右边部分括号里的，pFunParam为别名一：
```c++
typedef void (*pFunParam)();
```
再替换左边的变量b，pFunx为别名二：
```c++
typedef void (*pFunx)(pFunParam);
```
原声明的最简化版：
```c++
pFunx b[10];
```
---
理解复杂声明可用的“右左法则”：从变量名看起，先往右，再往左，碰到一个圆括号就调转阅读的方向；括号内分析完就跳出括号，还是按先右后左的顺序，如此循环，直到整个声明分析完。举例：
```c++
int (*func)(int *p);
```
首先找到变量名func，外面有一对圆括号，而且左边是一个`*`号，这说明func是一个指针；然后跳出这个圆括号，先看右边，又遇到圆括号，这说明`(*func)`是一个函数，所以func是一个指向这类函数的指针，即函数指针，这类函数具有`int*`类型的形参，返回值类型是int。
```c++
int (*func[5])(int *);
```
func右边是一个`[]`运算符，说明func是具有5个元素的数组；func的左边有一个`*`，说明func的元素是指针（注意这里的`*`不是修饰func，而是修饰`func[5]`的，原因是`[]`运算符优先级比`*`高，func先跟`[]`结合）。跳出这个括号，看右边，又遇到圆括号，说明func数组的元素是函数类型的指针，它指向的函数具有`int*`类型的形参，返回值类型为int。
也可以记住2个模式：
```c++
type (*)(....)函数指针
type (*)[]数组指针
```

+ typedef在语法上是一个存储类的关键字（如auto、extern、mutable、static、register等一样），虽然它并不真正影响对象的存储特性，如：
`typedef static int INT2; //不可行`

害，这东西，看看就行，知道有这么个神奇的函数指针和各种神奇的定义，然后可以用typedef简化不至于看不懂代码就好了。

#### 那我们开始了
```c++
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacer {
public: 
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
    :healthFunc(hcf){}
    int healthValue() const 
    {
        return healthFunc(*this);
    }
    ...
private:
    HealthCalcFunc healthFunc;
};
class EvilBadGuy: public GameCharacter{
public:
    explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalce)
    : GameCharacter(hcf) {...}
    ...
}
int loseHealthQuickly(const GameCharacter&);
int lostHealthSlowly(const GameCharacter&);

EvilBadGuy ebg1(loseHealthQuickly);
EvilBadGuy ebg2(loseHealthSlowly);

```

这提供了一些有趣的弹性：
同一人物类型的不同实体可以有不同的健康计算函数
计算函数可以在运行期变更

计算函数不再是继承体系中的成员函数意味着不需要特别访问对象内的特别成分，如果你需要这个non-public进行计算，就出现问题了。任何时候把class内的某个机能替换为class外部的某个等价机能都存在潜在的争议点。
一般而言唯一解决的办法就是弱化class的封装，比如声明non-member函数为friend，或是为实现的某一部分提供public访问函数。那么用函数指针替换virtual是否是优（上面两点）大过劣（可能必须降低封装性）的，则需要你自己分析。

### 藉由tr1::function完成Strategy模式
一旦习惯了template以及他们对隐式接口的使用，基于指针的做法看起来就有些苛刻和死板了。为什么必须是个函数呢？如果必须是函数，为什么一定返回int？为什么不能是个成员函数？
tr1::function可以解决我们的问题：
```c++
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacer {
public: 
    typedef std::tr1::function<int(const GameCharacter&)> HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
    :healthFunc(hcf){}
    int healthValue() const 
    {
        return healthFunc(*this);
    }
    ...
private:
    HealthCalcFunc healthFunc;
};

short calcHealth(const GameCharacter&);
struct HealthCaalculator{
    int operator()(const GameCharacter&)const{...}
};
class GameLevel{
public:
    float health(const GameCharacter&) const;
    ...
};
class EvilBadGuy: public GameCharacter{
    ...//同前
};
class EyeCandyCharacter: public GameCharacter {
    ...
};
EvilBadGuy ebg1(calcHealth);//函数

EyeCandyCharacter eccl(HealthCalculator());//函数对象

GameLevel currentLevel;
...
EvilBadGuy ebg2(                //成员函数
    std::tr1::bind(&GameLevel::health,
                   currentLevel,
                   _1)
);

```
在你发出感慨之前，我们先来看一下tr1::function，虽然现在已经变成std::function了。简单的讲就是泛化的函数指针。（其实就是我懒得去找function的资料了，可以遇见得到很长）
我们仔细来看ebg2。
首先ebg2要使用GameLevel的health。但是health他宣称接受一个参数，实际却需要两个参数，另一个是隐式参数GameLevel，也就是this指的那个。然而GameCharacters的计算函数只接受单一参数GameCharacter。所以如果我们用health，我们必须转换他。于是我们将currentLevel绑定为GameLevel对象，让它在「每次GameLevel::health被调用以计算ebg2的健康」时被使用。

### 古典的Strategy模式。
如果你对设计模式更感兴趣，那么传统的Strategy可能会吸引你：把健康计算函数做成分离的继承体系中的virtual函数，文字表述一下设计结果的话——
>GameCharacter是继承体系的根类，体系中的EvilBadGuy和EyeCandyCharacter都是继承类;HealthCalFunc是另一个继承体系的根类，体系中的SlowHealthLoser和Fast版本都是继承类;每一个GameCharacter对象都内含一个指针指向一个来自HealthCalcFunc继承体系的对象。

## 36：绝不重新定义继承而来的non-virtual函数
这里指向的是关于继承类和指针的奇异现象。
```c++
class B{
public:
    void mf();
    ...
};
class D: public B{
public:
    void mf();
    ...
};

D x;
B* pB = &x;
pB->mf();//B::mf()
D* pD = &x;
pD->mf();//D::mf();
```
此时D对象可能表现出B或者D的行为，取决于指向该对象的指针声明的类型。引用也同样有这样的问题。
造成这种行为的原因是，non-virtual函数是静态绑定的，virtual则是动态的。（我们下一条讲绑定的问题）

回顾32和34，我们知道：
+ 适用于B的每件事，也适用于D，因为每一个D都是一个B。
+ B的derived class一定会继承mf的接口和实现

所以如果D重新定义了mf，你的设计就出现了矛盾。

如果该条让你感到乏味，那可能是因为你读过了法则7，解释多态基类中的析构函数应该是virtual。本质而言，那只是该法则的一个特例罢了。

## 37:绝不重新定义继承而来的缺省参数值
依旧是例子起手
```c++
class Shape {
public:
    enum ShapeColor{ Red, Green, Blue };
    virtual void draw(ShapeColor color = Red) const = 0;
    ...
};
class Rectangle: public Shape{
public:
    virtual void draw(ShapeColor color = Green) const;//so bad
    ...
};
class Circle: public Shape{
public:
    virtual void draw(ShapeColor color) const;
};
Shape* ps;
Shape* pc = new Circle;
Shape* pr = new Rectangle;
ps = pc;//ps的动态类型是Circle*
ps = pr;//pr的动态类型是Rectangle*
```
所谓静态类型就是声明它时的类型，动态类型则是指目前所指对象的类型。而动态类型是可以改变的，如上的ps本来是无动态类型，后面发生了变化。
virtual由动态绑定而来，意思是调用哪个virtual函数是由对象的动态类型决定的；是的，你也许这些都懂，但缺省参数值却是静态绑定的，意即你可能在调用定义在继承类中的virtual函数的同时使用基类为它所指定的缺省参数值。

另外要注意客户使用Circle对象的draw时，一定要指定参数值。因为静态绑定下这个函数不会从基类继承缺省参数，但如果用指针或者引用就会继承从而可以不指定参数值（动态绑定）
```c++
pr->draw();//Raceangle::draw(Shape::Red)!
```
同时提供缺省给继承类和基类不是个好主意，因为代码重复，而且由于相依性，如果缺省参数改变那所有的都会改变。
最好的办法是考虑替代设计，比如NVI，因为NVI的virtual函数和实际函数是分离的，可以用非虚函数指定参数

## 38:通过复合塑模出has-a/根据某物实现出
复合(composition)这个词有许多同义词，包括分层(layering)、内含(containment)、聚合(aggregation)、内嵌(embedding)

之前说过public继承带有一种is-a（是一种）的意义，这里，复合也有类似的意义：有一个／根据某物实现出

比如你不会说人是一个名称、人是一个地址，你会说人有他们。
这很自觉，比较麻烦的是区分是一种和根据某物实现出。

你可以先认为它是一个is-a的关系，当你发现了一些致命的缺陷时，你应当去思考「根据实现」的可能，比如如果你使用list来实现Set，public继承是有问题的，因为list和set本身的理念有冲突的地方（重复元素）


## 39:明智而审慎的使用private继承
private继承在软件设计层面上没有意义，其意义只存在于软件实现层面。
private继承意味着根据某物实现出。所以尽可能的使用复合，必要时才使用private继承，主要是当protected成员或virtual函数牵涉进来的时候。
```c++
//比如我们有一个class叫Timer，有一个virtual函数叫onTick，即每次滴答
class Widget: private Timer{
private:
    virtual void onTick() const;
    ...
};
//===private继承并非绝对必要，如果我们以复合取而代之：
class Widget{
private:
    class WidgetTimer: public Timer{
    public:
        virtual void onTick() const;
        ...
    };
    WidgetTimer timer;
    ...
};
```
这是种在c++模拟「阻止继承类重新定义virtual函数」的能力的办法，就像java的final和c#的sealed。即是说在这个Widget的继承类里是不能重新定义onTick的。private继承却可以。
还有一个优点在于编译依存性——private继承Timer的还是很Timer的定义必须可见，所以必须include Timer.h。但如果WidgetTimer移出Widget之外而内含一个指针指向WidgetTimer，则只需要一个简单的WidgetTimer的声明式。这种解耦可能对大型系统很重要。

但private继承有一种适用情况：class不带任何数据时。这样的class没有non-static成员变量，没有virtual函数，也没有virtual base class。于是这样但empty class对象不使用任何空间，因为没有数据需要存储，然而由于技术上的理由，对象必须有非零大小（比如勒令一个char插入到空对象内）。
```c++
class Empty();
//---1----
class HoldsAnInt{
private:
    int x;
    Empty e;
};//sizeof(HoldsAnInt) > sizeof(int);
//---2----
class HoldsAnInt: private Empty {
private:
    int x;
};//sizeof(HoldsAnInt) == sizeof(int);
```
这是所谓EBO（empty base optimization，基类最优化）。
EBO只在单一继承下才可行。
这对致力于「对象尺寸最小化」的程序库开发者而言可能很重要。

现实中的「Empty」class往往并不真的是空的，往往内含typedef，enum，static成员变量，或non-virtual函数。

## 40:明智而审慎的使用多重继承
一旦涉及多重继承，c++社区便分为两个阵营：SI是好的，MI一定更好；和SI是好的，但MI不值得使用。
（MI：multiple inheritance多重继承；SI：single单一继承）

最先需要认清的一件事是，MI可能从一个以上的base继承相同名称，会导致歧义：
```c++
class B{
public:
    void ok();
};
class C{
private:
    void ok();
};
class A: public B,public C{ ... };
A a;
a.ok();
```
注意这里是存在歧义的，即使B和C的ok中只有一个是可用的。原因在于c++的名称查找优先于检验可用之前，所以会直接卡在两个相同匹配的名称而无法继续进行。所以你必须指定base。
所以指定成virtual base class（虚基类）是有必要的，这里是对virtual继承的忠告：
+ 非必要不要使用虚继承。
+ 如果使用virtual base class，尽可能避免在其中放置数据。

因为多重继承对virtual的依赖，这里也有几个忠告：
+ 如果能用多重继承实现，那么几乎一定有方案让单一继承行得通
+ 然而如果多重继承就是最简单合理的方法，那就用它。
+ virtual继承会增加大小、速度、初始化复杂度等成本。
+ 多重继承更复杂，导致新的歧义性和virtual的需要
+ public继承某个Interface class和private继承某个协助实现的class的组合是多重继承的一种正当用途。




