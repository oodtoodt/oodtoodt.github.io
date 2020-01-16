---
title: cpp-14笔记
date: 2020-01-14 16:05:58
tags:
- c++
- c++14
- company

categories:
- c++

---

先打预防针：14的内容不多。
<!--more-->

---

# C++14
应该说核心内容不多，如果你想去读官方1000页文档然后说不多那我无话可说。
这里就只捡取最流行的一份中文文档来说了
如果你打开cppreference，查看c++语言核心特性，会发现c++14只有一个变量模板

另外贴一个知乎大大的总结：
>最后总结一下就是，C++14可以称为更加完美的C++11，这是我的观点。我觉得这些提案本就应该在C++11出现，但是却由于各种因素才拖到C++14中了。 C++的确变得更加复杂了，但是对于今天学习C++的人来说是更幸福的，少了很多奇淫技巧，多了更多照顾开发者的东西。
——蓝色

## 一篇流传很广的文档：语言特性
### 泛型lambda，可以在形参中使用auto
```c++
auto adder = [](auto op1,auto op2) {return op1 + op2;};
```
### lambda捕捉初始化
c++14允许被捕捉的成员用任意的表达式初始化。
```c++
auto ptr = std::make_unique<int>(10);
auto lambda = [&ptr]{return *ptr};
//也是可以的了 auto lambda = [ptr = std::move(ptr)]{return *ptr;};
```
### auto推导
c++14的auto可以推导没有返回类型的函数了（之前必须显式声明，不过注意现在我们禁止带有歧义性的返回类型auto）
当decltype(auto)被用于声明变量时，该变量必须立即初始化。假设该变量的初始化表达式为e，那么该变量的类型将被推导为decltype(e)。也就是说在推导变量类型时，先用初始化表达式替换decltype(auto)当中的auto，然后再根据decltype的语法规则来确定变量的类型。
```c++
decltype(auto) g1() {return s.a;}
decltype(auto) i1 = a;
decltype(auto) i2 = std::move(a);
```

与auto一起做返回值的区别：
如果不推导出 void 返回类型的话，auto （可有 cv 限定符）一定会推导出返回类型为对象类型。并且应用数组到指针、函数到指针隐式转换。
auto 加上 & 或 && （可有 cv 限定符）一定会推导出返回类型为引用类型。
decltype(auto) 可以推导出对象类型，也可以推导出引用类型。具体取决于 decltype 应用到 return 语句中表达式的结果。

### 函数返回类型
使得没有return expression的函数也可以使用返回类型推导
如果你对之前的某篇文章很有印象的话，会记得有
```c++
auto f(int a, int b) -> decltype( a + b )
{
   int i = a + b;
   return i;
}
int main()
{
    std::cout << f(1,2) << std::endl;
    return 0;
}
```
对，返回类型后置，当然不是用在这里的，这里举个例子。
c++14之后，你就可以直接
```c++
auto f(int a, int b)
{
   int i = a + b;
   return i;
}
int main()
{
    std::cout << f(1,2) << std::endl;
    return 0;
}
```
注意，因为不允许歧义了，所以
```c++
auto sum(int i) {
  if (i == 1)
    return i;          // return type deduced to int
  else
    return sum(i-1)+i + 0.0; // 这样怎么办呢？
}
```
是不允许的

### 放松了constepr
放松了constexpr的概念，c++11中，constexpr函数只含有一个将返回的表达式（和包含static_assert声明在内的几个语句），c++14中则放松了这些限制
constexpr的函数将可以包含：
- 任何声明，除了static和thread_local变量、没有初始化的变量声明。
- 放松了constexpr的概念，c++11中，constexpr函数只含有一个将返回的表达式（和包含static_assert声明在内的几个语句），c++14中则放松了这些限制
constexpr的函数将可以包含：
- 任何声明，除了static和thread_local变量、没有初始化的变量声明。
- 条件分支if和switch，goto不行
- 所有的循环
- 表达式可以改变对象值，只需该对象生命期在常量表达式函数内开始
调用非constexpr函数仍然是受限的，所以如果使用基于范围的for，begin和end的重载必须自身被声明为constexpr，顺便一提，std::initializer_list在本地和全局都有这东西

所有被声明为constexpr的非静态成员函数也隐含声明为const（即函数不能修改*this的值）这一在c++11中定义的点在c++14中已经被删除。

多说几句，这个特性...
>这篇提案可谓是深得标准委员会人员的心，不可否认，这篇提案对于普通开发人员无疑是福音，因为可以再次提高性能。但是对于编译器开发人员简直是噩梦的一篇提案。说实话，在C++11的所有特性中，constexpr绝对是一个很难实现的特性，我不能透露ibm编译器如何实现的，感兴趣的可以去参看GCC或者Clang的实现。直到现在，Visual C++的实现constexpr都是Partial的状态。众所周至，constexpr有一些限定，比如只能返回一条语句等，即使这样，编译器做起来都是很头痛的一件事情。这篇提案简直逆天了，要去掉这些条件，还可以加入if，for什么的了，，具体的可以参看提案内容。我只能说简直丧心病狂，完全不管编译器开发者的死活。当时我们激烈讨论了到底怎么实现的，但是也没有讨论出来什么。但就是这篇提案，标准委员会举双手双脚赞成，而且还说有实现的先例：D语言。我也不知道该说什么了。
作者：蓝色
链接：https://www.zhihu.com/question/22765983/answer/22591090

### 变量模板
c++11引入了类型别名模板，c++14现在也可以创建变量模板
指using解决typedef在模板参数的问题
```c++
template<class T>
constexpr T pi = T(3.1415926535897932385);  // 变量模板
 
template<class T>
T circular_area(T r) // 函数模板
{
    return pi<T> * r * r; // pi<T> 是变量模板实例化
}
```
在类作用域中使用时，变量模板声明一个静态数据成员模板。
```c++
using namespace std::literals;
struct matrix_constants
{
    template<class T>
    using pauli = hermitian_matrix<T, 2>; // 别名模版
 
    template<class T> // 静态数据成员模板
    static constexpr pauli<T> sigmaX = { { 0, 1 }, { 1, 0 } }; 
 
    template<class T>
    static constexpr pauli<T> sigmaY = { { 0, -1i }, { 1i, 0 } };
 
    template<class T>
    static constexpr pauli<T> sigmaZ = { { 1, 0 }, { 0, -1 } };
};
```
除非变量模板被显式特化或显式实例化，否则它在特化时隐式实例化
```c++
struct limits {
    template<typename T>
    static const T min; // 静态数据成员模板的声明
};
template<typename T>
const T limits::min = { }; // 静态数据成员模板的定义
 
template<class T>
class X {
   static T s; // 类模板的非模板静态数据成员的声明
};
template<class T>
T X<T>::s = 0; // 类模板的非模板静态数据成员的定义
```
模板元编程离我们还有点距离，我们可以先放一放。

-----------------------------------------------
### 聚合体成员初始化。
c++11新增member initializer(即成员初始化列表)，之前不允许聚合体初始化，现在c++14可以了
聚合体：
aggregate:详见pod那个章节，这里我们稍微提一下。数组类型、结构体、联合体、类中，没有私有、保护的非静态数据成员、无用户提供的构造函数、无基类无虚函数的都是。
如果聚合体中间有嵌套，可以不用花括号分割。（`int arr[3],j;={1,2,3,4}`，j会分配4）

### 二进制字面值与数字分位符号
使用前缀0b或0B，引入'作为数字分位符号，使其具有更好的可读性。

### 属性：deprecated
指示声明有此属性的名字或实体被弃用，即允许但因故不鼓励使用。
语法
```c++
[[deprecated]]	(1)	
[[deprecated( 字符字面量 )]]	(2)	
```
字符字面量	-	能用于解释弃用的理由并/或提议代替用实体的文本

------------------------------------------

----------------------------------
## *新标准库的特性*
-----------

------
### 增加了共享的互斥体和锁
std::shared_timed_mutex

### 元函数别名
c++11定义了一组元函数用于查询类型是否具有某种特征，或者转换给定类型的某种特征
指的就是type_traits里的各种东西，以及别名模板。
我们先来一点前置知识学习
##### volatile
首先是c++11中的volatile
volatile:在柯林斯高级学习词典中这样解释：
A situation that is volatile is likely to change suddenly and unexpectedly.
注意，likely，suddenly，unexpectedly
总结来说，被 volatile 修饰的变量，在对其进行读写操作时，会引发一些可观测的副作用。而这些可观测的副作用，是由程序之外的因素决定的。
我们讨论c/c++的执行顺序时会提到——序列点。执行表达式有两种类型的动作：(1)计算某个值(2)副作用(例如访问volatile对象、原子同步、修改文件等)因此，如果两个表达式e1和e2中间有一个序列点，或者在c++中e1于序列中在e2之前，则e1的求值动作和副作用都会在e2的求值和副作用之前。
因此在c/c++中，对volatile对象的访问，有编译器优化上的副作用：
+ 不允许优化消失
+ 于序列上在另一个对volatile对象访问之前
（这里其实很像原子操作的某些规则，但是volatile访问的序列性是在单线程执行的前提）
##### 常见的错解
错解：用volatile修饰while的退出flag。应该用atomic
volatile 本质是让编译器在每次遇到被它修饰的变量时，不能根据代码上下文假设它的状态。注意是「让『编译器』」。所以编译器不会对它做某些类型的优化，也不会交换两个 volatile 访问的顺序。但是这不保证 CPU 执行时不会交换顺序。
但是还是要结合一下cpu的情况，根据具体环境给一个可用不可用的范围约束。

好，现在我们学会了volatile，于是我们来看remove_cv
##### remove_cv
很简单！去掉const和volatile前缀，其实就是remove_const和remove_volatile的结合
然后呢，我们看到了type_traits头文件！
想到什么了吗！
之前没讲的，类型萃取！它leile！
#### 类型萃取

std::decay对类型进行退化处理，退化掉引用和cv符。
如果`is_array<U>::value`为真，则修改类型为`remove_reference<U>::type*`
否则，如果`is_function < U > ::value`为真，修改类型为`add_pointer< U >::type`
否则，修改类型为`remove_cv< U >::type`
可以方便的获得函数指针
std::result_of元函数可以获取调用对象的返回类型
enable_if是利用SFINAE根据条件重载函数

#### ...这和c++14有什么关系呢

但是在c++11中，它们用在模板中时，必须用typename关键字
c++14中提供了更便捷的写法————利用类型别名模板。规则：如果标准库的某个类模板只含有有唯一的成员，即成员类型type，那么提供`std::some_class_t<T>`作为`typename std::some_class::type`的别名
稍微罗列几个就懂了：
>`remove_const,remove_volatile,remove_cv,add_const,add_reference,add_lvalue_reference(r),make_signed,make_unsigned,remove_extent,remove_all_extents,remove_pointer,aligned_storage,decay,enable_if,conditional,common_type,underlying_type,result_of,tuple_element`

对不起发现了几个很有趣的类型就都写上了。
```c++
template<class T>
type_object<std::remove_cv_t<std::remove_reference_t<T>>>get_type_object(T&);
```
唔，似乎没说明到底是啥ww，具体就是给一个预定义的using来用
```c++
template <class T>
struct remove_reference;

template <class T>
using remove_reference_t = typename remove_reference<T>::type;
```
----------------------------------------------------

### 关联容器的异构查找
即定义了operator<的都可以作为set等查找类型
strict weak orderings，就
```c++
if ( x < y ) { return true;} if ( y < x ) {return false;}
```
具体请查标准库文档
比如const char* 与string
标准库的泛型比较器都支持这一点
如果不知道你根本不会相信以前是不支持这个东西的（指复杂度会远超logn

### 新增自定义字面量特性（标准库）
**"s"，用于创建各种std::basic_string类型
"h","min","s","ms","us","ns"用于创建相应的std::chrono::duration时间间隔**

### 通过类型寻址多元组
```c++
tuple<string,string,int>t("foo","bar",7);
int i = get<int>(t);//i == 7
int j = get<2>(t)//same as before j == 7;
string s = get<string>(t);//ce due to ambiguity
```
### 较小的标准库特性
std::make_unique可以像std::make_shared一样使用产生std::unique_ptr对象（？为什么不在c++11里加这个）
std::is_final，用于识别一个class类型是否禁止被继承
std::integral_constant增加了一个返回常量值的operator()
全局std::begin/std::end函数之外，增加了std::cbegin/std::cend函数，它们总是返回常量迭代器

### c++14移除或抛弃的特性
被c++14移除的标准特性（这里指一些可能本来会在c++14出现的东西
- 关于数组的扩展
在c++11和之前的标准中，在堆栈上分配的数组被限制为拥有一个固定的、编译器确定的长度。这一扩展允许堆栈上分配的一个数组最后一维拥有运行期确定的长度。
添加了std::dynarray类型，它拥有与std::vector和std::array类似的接口，代表一个固定长度的数组。被明显地设计为当它被放置在栈上时，可以使用栈内存而不是堆内存
在斟酌之后，它被从c++14中移除了。
- optional值
类似于c#的可空类型，也没采用
- concepts lite
回炉重造了

### 拾遗
#### std::exchange
```c++
template< class T, class U = T >
T exchange( T& obj, U&& new_value );
```
以 new_value 替换 obj 的值，并返回 obj 的旧值。

#### std::integer_sequence
```c++
template< class T, T... Ints >
struct integer_sequence;
//(C++14 起)
```
类模板 std::integer_sequence 表示一个编译时的整数序列。在用作函数模板的实参时，能推导参数包 Ints 并将它用于包展开。

### 结语
c++14结束啦。
本期内容来自各种网上资料与一篇流传甚广的ppt...
补这篇文档的时候找到了不少有趣的资料，感觉很不错。
下篇就是c++17了，17的资料真的不多，争取明天弄完吧——

