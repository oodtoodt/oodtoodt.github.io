---
title: EffectiveModernCpp-note
date: 2020-05-07 15:31:23
tags:
---

真绝了，没想到我又回来了学c++了
<!--more-->

---
## item5：优先考虑auto而非显式
auto好处多多
+ 可以简化语法，减少重复量
+ 更适合存储闭包
+ 必须初始化
+ 避免依赖类型快捷方式的问题
```c++
std::vector<int> v;
unsigned sz = v.size();
auto sz = v.size();//better
```
因为实际上，size的返回值是`std::vector<int>::size_type`，在32-bit和64-bit的windows上并不一定是相同的。auto保证了不会有问题
+ auto保证避免很难意识到的类型不匹配的错误
```c++
std::unordered_map<std::sring,int> m;
for(const std::pair<std::string,int>&p : m) {}//wrong！
for(const auto & p : m) {}
```
因为unordered_map的key是一个常量，所以应当是`std::pair<const std::string,int>`，但这真的很难发现，也真的很难避免，除非你无所不知（指知道）。


## item6：当auto推导出非预期类型时应使用显式类型初始化
这个就很离谱，我用auto是为了什么？
那么问题的原因是什么呢，是一些代理类会占据auto使得一些令人困惑的事情发生，导致一些可能原先有的，指向某些东西的指针悬挂。
解决就是显式初始化。

## Item7：区别使用()和{}创建对象
```c++
Widget w1(10)
Widget w2();
```
我这样放在一起，你应当会觉得它们都是初始化，但其实不是，后者会变成函数声明。
大括号则没有这种问题。
大括号还能禁止窄变换，就是隐式自动变换，`int sum{1.0+2.0}`
大括号的问题是会被`std::initializer_list<>`劫持，认为整个大括号内的都是一套里的东西，这有点像auto去识别大括号。（当然，必须是`<>`中的实参可以转化的情况）
这里可以提一嘴：
```c++
std::vector<int> v1(10,20);
std::vector<int> v2{10,20};
```
前者是10个元素都是20的vector，后者是元素值分别为10和20的vector。

## Item8：优先考虑nullptr而非0/null
因为NULL的实现细节有不确定因素，是除int外的整形。
nullptr的优点是它不是整形
也不是指针类型，但可以认为是通用类型的指针，真正类型是std::nullptr_t，在一个完美循环定义后，std::nullptr_t又被定义为nullptr（？这里我持怀疑态度）
因此nullptr可以避开函数重载导致的种种问题，以及模板的识别问题

## Item9：优先考虑别名声明而非typedefs
这一点其实很常规，但是有值得深究的理由
首先在模板中用using非常容易，用typedef你就得套一层struct然后实际使用起来还很麻烦：如果使用typedef声明一个持有链表的对象，对象又用了模板参数，你就不得不加上typename，这里涉及的类型依赖可以去查阅自己写的笔记以及挖坑不填的cpptt
所以就有了`std::remove_const<T>::type -> std::remove_const_t<T>`的变化，怎么样，是不是串起来了？没错，标准委员会背锅。

## Item10：优先考虑限域枚举而非未限域枚举
即传统的enum {}不如现代的enum class {}，具体说前者有这些问题：
+ 枚举名泄漏
+ 枚举名会隐式转化为整形（/浮点型）
+ 你可能不记得前置声明的事宜
第三个不算问题，只是提到了。这里我想贴一下他提到的这个传统enum用到的地方，以及如何优化限域枚举的表现：
假设我们有一个c++11的tuples保存了用户的名字、email、声誉点
```c++
using UserInfo = 
    std::tuple<std::string,
    std::string,
    std::size_t> ;
UserInfo uInfo;
...
auto val = std::get<1>(uInfo);
```
作为一个程序员，你应该记住第一个字段代表用户的email吗？不。
可以使用非限域枚举关联来避免需求
```c++
enum UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo;
...
auto val = std::get<uiEmail>(uInfo);
```
因为枚举名被隐式转换成std::size_t了，没想到吧.jpg
如果你用限定的话就必须自己来动手
```c++
auto val = std::get<static_cast<std::size_t>(UserInfoFields::uiEmail)>(uInfo);
```
作为一个程序员，你应当想到可以写一个函数传入枚举名并返回对应的std::size_t值。
std::get是一个模板，你给出的std::size_t值是模板实参(`即<>而非()`)因此枚举名变换会发生在编译期，所以这必须是constexpr模板函数
如果我们想要更泛化的使用它，泛化其返回类型，则可以通过std::underlying_type,最后可以在编译期接受任意枚举名并返回它的值
```cpp
template<typename E>
constexpr typename std::underlying_type<E>::type
  toUType(E enumerator) noexcept
{
  return
    static_cast<typename
      std::underlying_type<E>::type>(enumerator);
}
```

在C++14中，**toUType**还可以进一步用`std::underlying_type_t`（参见Item 9）代替`typename std::underlying_type<E>::type`打磨：
```cpp
template<typename E> // C++14
constexpr std::underlying_type_t<E>
  toUType(E enumerator) noexcept
{
  return static_cast<std::underlying_type_t<E>>(enumerator);
}
```
还可以再用C++14 auto（参见Item 3）打磨一下代码：
```cpp
template<typename E> // C++14
constexpr auto
  toUType(E enumerator) noexcept
{
  return static_cast<std::underlying_type_t<E>>(enumerator);
}
```
不管它怎么写，**toUType**现在允许这样访问tuple的字段了：
```cpp
auto val = std::get<toUType(UserInfoFields::uiEmail)>(uInfo);
```

## Item11：优先考虑使用deleted函数而非使用未定义的私有声明
在effective c++中，我们就提到过防止客户调用某些自动声明的函数的方法是把它们声明为私有成员函数，见原书item6
任何函数都能delete，包括非成员函数和模板实例，这些在private声明中做不到。

## Item12：使用override声明重写函数
...好像没什么好说的，就这么干就行
用成员函数限定 `&  &&`来区分左值对象和右值对象

## Item13：优先考虑const_iterator而非iterator
c++98中const_iterator非常麻烦，是万恶之源（？）
c++11没有cbegin,cend,rbegin,rend...你也许要自己实现：
```c++
template <class C>
auto cbegin(const C& container)->decltype(std::begin(container))
{
	return std::begin(container); // 解释见下
}
```

## Item14：如果函数不抛出异常就用noexcept
优化性而言，是`noexcept>throw()>无`
noexcept是函数接口的一部分，这意味着调用者会依赖它、
noexcept函数较之于非noexcept函数更容易优化
noexcept对于移动语义,swap，内存释放函数和析构函数非常有用（如何知道一个函数中的移动操作是否产生异常？答案很明显：它检查是否声明noexcept。）
大多数函数是异常中立的(译注：可能抛也可能不抛异常）而不是noexcept

## Item15：尽可能使用constexpr
如果要颁一个C++11中最令人困惑的词的奖，constexpr可能会赢得这个奖
尽可能的使用constexpr表示你需要长期坚持对某个对象或者函数施加这种限制。

这里提到的基本上都是常规的constexpr，但是作者指出：如果constexpr被一个或者多个编译器不可知的值调用时，它会运行时计算它的结果。这跟编译期计算有点相悖，我们留个心眼在这里。或者回去看看其他的翻译版本

## Item16：让const成员函数线程安全
这篇我就没看懂。虽然原子量和线程的东西确实是这样，两个操作量尽量用互斥锁，我懂了，但是和const成员函数有啥关系。。。
就是const里面如果有mutable还是会出现线程的危险

## Item17：理解特殊成员函数的生成
Rule of Three：这个规则告诉我们如果你声明了拷贝构造函数，拷贝赋值运算符，或者析构函数三者之一，你应该也声明其余两个
>两个拷贝操作是独立的：声明一个不会限制编译器声明另一个。所以如果你声明一个拷贝构造哈说，但是没有声明拷贝赋值运算符，如果写的代码用到了拷贝赋值，编译器会帮助你生成拷贝赋值运算符重载。同样的，如果你声明拷贝赋值运算符但是没有拷贝构造，代码用到拷贝构造编译器就会生成它。上述规则在C++98和C++11中都成立。
>再进一步，如果一个类显式声明了拷贝操作，编译器就不会生成移动操作。这种限制的解释是如果声明拷贝操作就暗示着默认逐成员拷贝操作不适用于该类，编译器会明白如果默认拷贝不适用于该类，移动操作也可能是不适用的。

Rule of Three带来的后果就是只要出现用户定义的析构函数就意味着简单的逐成员拷贝操作不适用于该类。接着，如果一个类声明了析构也意味着拷贝操作可能不应该自定生成，因为它们做的事情可能是错误的。在C++98提出的时候，上述推理没有得倒足够的重视，所以C++98用户声明析构不会左右编译器生成拷贝操作的意愿。C++11中情况仍然如此，但仅仅是因为限制拷贝操作生成的条件会破坏老代码。

Rule of Three规则背后的解释依然有效，再加上对声明拷贝操作阻止移动操作隐式生成的观察，使得C++11不会为那些有用户定义的析构函数的类生成移动操作。所以仅当下面条件成立时才会生成移动操作：
类中没有拷贝操作
类中没有移动操作
类中没有用户定义的析构


假设编译器生成的函数行为是正确的（即逐成员拷贝类数据是你期望的行为），你的工作很简单，C++11的=default就可以表达你想做的
然而用户声明的析构函数会抑制编译器生成移动操作，所以如果该类需要具有移动性，就为移动操作加上=default。声明移动会抑制拷贝生成，所以如果拷贝性也需要支持，再为拷贝操作加上=default：
问题就在于比如你如果不加，然后某一天突然加上了析构函数（或者其他什么）阻止了默认移动函数的构造，你仍能通过编译，并正确运行。但这时使用的是拷贝构造函数，完全没有移动的性能优势了。
