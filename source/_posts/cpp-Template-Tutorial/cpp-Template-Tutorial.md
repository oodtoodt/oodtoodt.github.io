---
title: cpp-Template-Tutorial
date: 2020-01-19 15:23:51
tags:
- c++
- template
- home

categories:
- c++
---

#
归根结底，模板无外乎两点：
+ 函数或者类里面，有一些类型我们希望它能变化一下，我们用标识符来代替它，这就是“模板参数”；
+ 在需要这些类型的地方，写上相对应的标识符（“模板参数”）。

在学习模板的时候，要反复做以下的思考和练习：

1. 提出问题：我的需求能不能用模板来解决？
2. 怎么解决？
3. 把解决方案用代码写出来。
4. 如果失败了，找到原因。是知识有盲点（例如不知道怎么将 T& 转化成 T），还是不可行（比如试图利用浮点常量特化模板类，但实际上这样做是不可行的）？

通过重复以上的练习，应该可以对模板的语法和含义都有所掌握。如果提出问题本身有困难，或许下面这个经典案例可以作为你思考的开始：

+ 写一个泛型的数据结构：例如，线性表，数组，链表，二叉树；
+ 写一个可以在不同数据结构、不同的元素类型上工作的泛型函数，例如求和；


## 编译器推导
是可以推导类型的
但是不能推导返回类型（毕竟返回的类型不会扔给后面的模板里）

## 整型参数
最基本的功能是定义一个常数。
注意整型参数必须能被在编译器推导出来，不然出错

```c++
template <int i> class A 
{
public:
    void foo(int)
    {
    }
};
template <uint8_t a, typename b, void* c> class B {};
template <bool, void (*a)()> class C {};
template <void (A<3>::*a)(int)> class D {};

template <int i> int Add(int a)	// 当然也能用于函数模板
{
    return a + i;
}

void foo()
{
    A<5> a;
    B<7, A<5>, nullptr>	b; // 模板参数可以是一个无符号八位整数，可以是模板生成的类；可以是一个指针。
    C<false, &foo> c;      // 模板参数可以是一个bool类型的常量，甚至可以是一个函数指针。
    D<&A<3>::foo> d;       // 丧心病狂啊！它还能是一个成员函数指针！
    int x = Add<3>(5);     // x == 8。因为整型模板参数无法从函数参数获得，所以只能是手工指定啦。
}

template <float a> class E {}; // ERROR: 别闹！早说过只能是整数类型的啦！
```

## 特化
```c++
class ClassB {};

template <> class TypeToID<void ()>;      // 函数的TypeID
template <> class TypeToID<int[3]>;       // 数组的TypeID
template <> class TypeToID<int (int[3])>; // 这是以数组为参数的函数的TypeID
template <> class TypeToID<int (ClassB::*[3])(void*, float[2])>; // 我也不知道这是什么了，自己看着办吧。

//甚至连 const 和 volatile 都能装进去：

template <> class TypeToID<int const * volatile * const volatile>;
```
当然啦，这里我们实现的不算是真正的 RemovePointer，因为我们只去掉了一层指针。而如果传进来的是类似` RemovePointer<int**> `这样的东西呢？是的没错，去掉一层之后还是一个指针。`RemovePointer<int**>::Result `应该是一个 `int*`，要怎么才能实现我们想要的呢？聪明的你一定能想到：只要像剥洋葱一样，一层一层一层地剥开，不就好了吗！相应地我们应该怎么实现呢？可以把 RemovePointer 的特化版本改成这样（当然如果有一些不明白的地方你可以暂时跳过，接着往下看，很快就会明白的）：
```c++
template <typename T>
class RemovePointer<T*>
{
public:
    // 如果是传进来的是一个指针，我们就剥夺一层，直到指针形式不存在为止。
    // 例如 RemovePointer<int**>，Result 是 RemovePointer<int*>::Result，
    // 而 RemovePointer<int*>::Result 又是 int，最终就变成了我们想要的 int，其它也是类似。
    typedef typename RemovePointer<T>::Result Result;
};
```
恍然大悟！
```c++
using a = typename RemovePointer<T>::a;
```


那为什么` int*` 就会找 `int*，float *`因为没有合适的特化就去找 T*，更一般的就去找 T 呢？废话，有专门为你准备的东西你不用，非要自己找事？这就是直觉。 但是呢，直觉对付更加复杂的问题还是没用的（也不是没用，主要是你没这个直觉了）。我们要把这个直觉，转换成合理的规则——即模板的匹配规则。 当然，这个匹配规则是对复杂问题用的，所以我们会到实在一眼看不出来的时候才会动用它。一开始我们只要把握：模板是从最特殊到最一般形式进行匹配的 就可以了。

## 
```c++
template <typename T> struct X {};
	
template <typename T> struct Y
{
    typedef X<T> ReboundType;						// 这里为什么是正确的？
    typedef typename X<T>::MemberType MemberType2;	// 这里的typename是做什么的？
    typedef UnknownType MemberType3;				// 这里为什么会出错？
};
```
我想大家尽管不能处理所有名称查找中所遇到的问题，但是对一些常见的名称查找规则也有了充分的经验，可以解决一些常见的问题。 但是模板的引入，使得名称查找这一本来就不简单的基本问题变得更加复杂了。 考虑下面这个例子：
```c++
struct A  { int a; };
struct AB { int a, b; };
struct C  { int c; };

template <typename T> foo(T& v0, C& v1){
    v0.a = 1;
    v1.a = 2;
    v1.c = 3;
}
```
简单分析上述代码很容易得到以下结论：

函数foo中的变量v1已经确定是struct C的实例，所以，v1.a = 2;会导致编译错误，v1.c = 3;是正确的代码；
对于变量v0来说，这个问题就变得很微妙。如果v0是struct A或者struct AB的实例，那么foo中的语句v0.a = 1;就是正确的。如果是struct C，那么这段代码就是错误的。
因此在模板定义的地方进行语义分析，并不能完全得出代码是正确或者错误的结论，只有到了实例化阶段，确定了模板参数的类型后，才知道这段代码正确与否。令人高兴的是，在这一问题上，我们和C++标准委员会的见地一致，说明我们的C++水平已经和Herb Sutter不分伯仲了。既然我们和Herb Sutter水平差不多，那凭什么人家就吃香喝辣？下面我们来选几条标准看看服不服：

>14.6 名称解析（Name resolution）
1)模板定义中能够出现以下三类名称：
模板名称、或模板实现中所定义的名称；
和模板参数有关的名称；
模板定义所在的定义域内能看到的名称。
…
9)… 如果名字查找和模板参数有关，那么查找会延期到模板参数全都确定的时候。 …
10)如果（模板定义内出现的）名字和模板参数无关，那么在模板定义处，就应该找得到这个名字的声明。…
14.6.2 依赖性名称（Dependent names）
1)…（模板定义中的）表达式和类型可能会依赖于模板参数，并且模板参数会影响到名称查找的作用域 … 如果表达式中有操作数依赖于模板参数，那么整个表达式都依赖于模板参数，名称查找延期到模板实例化时进行。并且定义时和实例化时的上下文都会参与名称查找。（依赖性）表达式可以分为类型依赖（类型指模板参数的类型）或值依赖。
14.6.2.2 类型依赖的表达式
2)如果成员函数所属的类型是和模板参数有关的，那么这个成员函数中的this就认为是类型依赖的。
14.6.3 非依赖性名称（Non-dependent names）
1)非依赖性名称在模板定义时使用通常的名称查找规则进行名称查找。
Working Draft: Standard of Programming Language C++, N3337(http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3337.pdf)

知道差距在哪了吗：人家会说黑话。什么时候咱们也会说黑话了，就是标准委员会成员了，反正懂得也不比他们少。不过黑话确实不太好懂 —— 怪我翻译不好的人，自己看原文，再说好懂了人家还靠什么吃饭 —— 我们来举一个例子：
```c++
int a;
struct B { int v; }
template <typename T> struct X {
    B b;                  // B 是第三类名字，b 是第一类
    T t;                  // T 是第二类
    X* anthor;            // X 这里代指 X<T>，第一类
    typedef int Y;        // int 是第三类
    Y y;                  // Y 是第一类
    C c;                  // C 什么都不是，编译错误。
    void foo() {
       b.v += y;          // b 是第一类，非依赖性名称
       b.v *= T::s_mem;   // T::s_mem 是第二类
                          // s_mem的作用域由T决定
                          // 依赖性名称，类型依赖
    }
};
```
所以，按照标准的意思，名称查找会在模板定义和实例化时各做一次，分别处理非依赖性名称和依赖性名称的查找。这就是“两阶段名称查找”这一名词的由来。只不过这个术语我也不知道是谁发明的，它并没有出现的标准上，但是频繁出现在StackOverflow和Blog上。

```c++
template <typename T> struct Y
{
    // X可以查找到原型；
    // X<T>是一个依赖性名称，模板定义阶段并不管X<T>是不是正确的。
    typedef X<T> ReboundType;
	
    // X可以查找到原型；
    // X<T>是一个依赖性名称，X<T>::MemberType也是一个依赖性名称；
    // 所以模板声明时也不会管X模板里面有没有MemberType这回事。
    typedef typename X<T>::MemberType MemberType2;
	
    // UnknownType 不是一个依赖性名称
    // 而且这个名字在当前作用域中不存在，所以直接报错。
    typedef UnknownType MemberType3;				
};
```
下面，唯一的问题就是第二个：typename是做什么的？

对于用户来说，这其实是一个语法噪音。也就是说，其实就算没有它，语法上也说得过去。事实上，某些情况下MSVC的确会在标准需要的时候，不用写typename。但是标准中还是规定了形如 T::MemberType 这样的qualified id 在默认情况下不是一个类型，而是解释为T的一个成员变量MemberType，只有当typename修饰之后才能作为类型出现。

事实上，标准对typename的使用规定极为复杂，也算是整个模板中的难点之一。如果想了解所有的标准，需要阅读标准14.6节下2-7条，以及14.6.2.1第一条中对于current instantiation的解释。

简单来说，**如果编译器能在出现的时候知道它是一个类型，那么就不需要typename，如果必须要到实例化的时候才能知道它是不是合法，那么定义的时候就把这个名称作为变量而不是类型。**

我们用一行代码来说明这个问题：
```c++
a * b;
```
在没有模板的情况下，这个语句有两种可能的意思：如果a是一个类型，这就是定义了一个指针b，它拥有类型`a*`；如果a是一个对象或引用，这就是计算一个表达式`a*b`，虽然结果并没有保存下来。可是如果上面的a是模板参数的成员，会发生什么呢？
```c++
template <typename T> void meow()
{
    T::a * b; // 这是指针定义还是表达式语句？
}
```
编译器对模板进行语法检查的时候，必须要知道上面那一行到底是个什么——这当然可以推迟到实例化的时候进行（比如VC，这也是上面说过VC可以不加typename的原因），不过那是另一个故事了——显然在模板定义的时候，编译器并不能妄断。因此，C++标准规定，在没有typename约束的情况下认为这里`T::a`不是类型，因此`T::a * b`; 会被当作表达式语句（例如乘法）；而为了告诉编译器这是一个指针的定义，我们必须在`T::a`之前加上typename关键字，告诉编译器T::a是一个类型，这样整个语句才能符合指针定义的语法。

在这里，我举几个例子帮助大家理解typename的用法，这几个例子已经足以涵盖日常使用（预览）：
```c++
struct A;
template <typename T> struct B;
template <typename T> struct X {
    typedef X<T> _A; // 编译器当然知道 X<T> 是一个类型。
    typedef X    _B; // X 等价于 X<T> 的缩写
    typedef T    _C; // T 不是一个类型还玩毛
    
    // ！！！注意我要变形了！！！
    class Y {
        typedef X<T>     _D;          // X 的内部，既然外部高枕无忧，内部更不用说了
        typedef X<T>::Y  _E;          // 嗯，这里也没问题，编译器知道Y就是当前的类型，
                                      // 这里在VS2015上会有错，需要添加 typename，
                                      // Clang 上顺利通过。
        typedef typename X<T*>::Y _F; // 这个居然要加 typename！
                                      // 因为，X<T*>和X<T>不一样哦，
                                      // 它可能会在实例化的时候被别的偏特化给抢过去实现了。
    };
    
    typedef A _G;                   // 嗯，没问题，A在外面声明啦
    typedef B<T> _H;                // B<T>也是一个类型
    typedef typename B<T>::type _I; // 嗯，因为不知道B<T>::type的信息，
                                    // 所以需要typename
    typedef B<int>::type _J;        // B<int> 不依赖模板参数，
                                    // 所以编译器直接就实例化（instantiate）了
                                    // 但是这个时候，B并没有被实现，所以就出错了
};
```

### 偏特化的另一个例子 
```c++
template <typename... Ts, typename U> class X {};              // (1) error!
template <typename... Ts>             class Y {};              // (2)
template <typename... Ts, typename U> class Y<U, Ts...> {};    // (3)，这里就是偏特化
template <typename... Ts, typename U> class Y<Ts..., U> {};    // (4) error!
```
偏特化时，模板参数列表并不代表匹配顺序，它们只是为偏特化的模式提供的声明，也就是说，它们的匹配顺序，只是按照`<U, Ts...>`来，而之前的参数只是告诉你Ts是一个类型列表，而U是一个类型，排名不分先后。


## 一个例子(模板的默认实参)
这中间的细节和思想我觉得都很有用，全部贴上来了。

在上一节中，我们介绍了模板对默认实参的支持。当时我们的例子很简单，默认模板实参是一个确定的类型void或者自定义的null_type：
```c++
template <
    typename T0, typename T1 = void, typename T2 = void
> class Tuple;
```
实际上，模板的默认参数不仅仅可以是一个确定的类型，它还能是以其他类型为参数的一个类型表达式。 考虑下面的例子：我们要执行两个同类型变量的除法，它对浮点、整数和其他类型分别采取不同的措施。 对于浮点，执行内置除法；对于整数，要处理除零保护，防止引发异常；对于其他类型，执行一个叫做CustomeDiv的函数。

第一步，我们先把浮点正确的写出来：
```c++
#include <type_traits>

template <typename T> T CustomDiv(T lhs, T rhs) {
    // Custom Div的实现
}

template <typename T, bool IsFloat = std::is_floating_point<T>::value> struct SafeDivide {
    static T Do(T lhs, T rhs) {
        return CustomDiv(lhs, rhs);
    }
};

template <typename T> struct SafeDivide<T, true>{    // 偏特化A
    static T Do(T lhs, T rhs){
        return lhs/rhs;
    }
};

template <typename T> struct SafeDivide<T, false>{   // 偏特化B
    static T Do(T lhs, T rhs){
        return lhs;
    }
};

void foo(){
    SafeDivide<float>::Do(1.0f, 2.0f);	// 调用偏特化A
    SafeDivide<int>::Do(1, 2);          // 调用偏特化B
}
```
在实例化的时候，尽管我们只为SafeDivide指定了参数T，但是它的另一个参数IsFloat在缺省的情况下，可以根据T，求出表达式`std::is_floating_point<T>::value`的值作为实参的值，带入到SafeDivide的匹配中。

嗯，这个时候我们要再把整型和其他类型纳入进来，无外乎就是加这么一个参数：
```c++
#include <complex>
#include <type_traits>

template <typename T> T CustomDiv(T lhs, T rhs) {
    T v;
    // Custom Div的实现
    return v;
}

template <
    typename T,
    bool IsFloat = std::is_floating_point<T>::value,
    bool IsIntegral = std::is_integral<T>::value
> struct SafeDivide {
    static T Do(T lhs, T rhs) {
        return CustomDiv(lhs, rhs);
    }
};

template <typename T> struct SafeDivide<T, true, false>{    // 偏特化A
    static T Do(T lhs, T rhs){
        return lhs/rhs;
    }
};

template <typename T> struct SafeDivide<T, false, true>{   // 偏特化B
    static T Do(T lhs, T rhs){
        return rhs == 0 ? 0 : lhs/rhs;
    }
};

void foo(){
    SafeDivide<float>::Do(1.0f, 2.0f);	                          // 调用偏特化A
    SafeDivide<int>::Do(1, 2);                                    // 调用偏特化B
    SafeDivide<std::complex<float>>::Do({1.f, 2.f}, {1.f, -2.f}); // 调用一般形式
}
```
当然，这时也许你会注意到，is_integral，is_floating_point和其他类类型三者是互斥的，那能不能只使用一个条件量来进行分派呢？答案当然是可以的：goo.gl/jYp5J2：
```c++
#include <complex>
#include <type_traits>

template <typename T> T CustomDiv(T lhs, T rhs) {
    T v;
    // Custom Div的实现
    return v;
}

template <typename T, typename Enabled = std::true_type> struct SafeDivide {
    static T Do(T lhs, T rhs) {
        return CustomDiv(lhs, rhs);
    }
};

template <typename T> struct SafeDivide<
    T, typename std::is_floating_point<T>::type>{    // 偏特化A
    static T Do(T lhs, T rhs){
        return lhs/rhs;
    }
};

template <typename T> struct SafeDivide<
    T, typename std::is_integral<T>::type>{          // 偏特化B
    static T Do(T lhs, T rhs){
        return rhs == 0 ? 0 : lhs/rhs;
    }
};

void foo(){
    SafeDivide<float>::Do(1.0f, 2.0f);	// 调用偏特化A
    SafeDivide<int>::Do(1, 2);          // 调用偏特化B
    SafeDivide<std::complex<float>>::Do({1.f, 2.f}, {1.f, -2.f});
}
```
我们借助这个例子，帮助大家理解一下这个结构是怎么工作的：

对`SafeDivide<int>`
通过匹配类模板的泛化形式，计算默认实参，可以知道我们要匹配的模板实参是`SafeDivide<int, true_type>`

计算两个偏特化的形式的匹配：A得到`<int, false_type>`,和B得到` <int, true_type>`

最后偏特化B的匹配结果和模板实参一致，使用它。

针对`SafeDivide<complex<float>>`
通过匹配类模板的泛化形式，可以知道我们要匹配的模板实参是`SafeDivide<complex<float>, true_type>`

计算两个偏特化形式的匹配：A和B均得到`SafeDivide<complex<float>, false_type>`

A和B都与模板实参无法匹配，所以使用原型，调用CustomDiv

## 后悔药——