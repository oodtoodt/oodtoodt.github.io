---
title: sol2笔记
date: 2020-01-13 17:16:15
tags:
- lua
- company
- sol2

categories:
- lua
---

sol2是一个轻量的用于C+绑定Lua脚本的库，仅由头文件组成，方便集成，并提供了大量易用的API接口，可以便利地将Lua脚本与C+代码绑定起来，而不必去关心如何使用那些晦涩的Lua C API。
<!--more-->

---
哦既然写到了sol2，那么也写一下lua的。
找了半天没找到个合适的说明

摘一段知乎的吧
    Python有明确的OOP系统，使用方式一致，便于创建各种库和进行分享（OOP对复用来说是决定性质的）。而Lua的优势也是它的弱势，它太自由了，我们用一个第三方Lua库，几乎只能使用它，而很难基于它进行二次封装和开发，你可以看LuaOOP里面有多少种，还不提没写在里面的。定义方式无法统一，就很难做反射，编辑器也没法针对每种实现给出提示。  
    Lua现在活跃的场所，都是平台本身或者第三方加入了支持后，能够以确定方式使用的场景，这也是Lua自身定位，所以它大多使用在游戏内，或者软件内，导致它基本是作为内部项目使用，不会有什么曝光度。而Python的定位是独立语言的，这也决定它受限更小，应用场景更大。  
    单纯论语言的话，我还是更喜欢Lua，语法设计太优秀了  
    作者：Kurapica
    链接：https://www.zhihu.com/question/273743493/answer/370642314
    来源：知乎
    著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

另外似乎我们这边很看重lua比起c++来说的高安全性

废话有点多了，笔记，来!

---

## sol2笔记
范哥nb！
今天跳过各种c调用lua的api，进入sol2学习。
### 入门
首先惯例（起手
```c++
sol::state lua;
lua.open_libraries(sol::lib::base);
```
### 运行一个脚本
```c++
lua.script();
lua.script_file();//注意运行lua代码要检查
auto bad_code_result = lua.script("..",[](lua_State*, sol::protected_function_result pfr)
c_assert(!bad_code_result.valid());
//使用.safe_script来看到更多的安全方法
```
### 传参
传参

```c++
//用...访问
const auto& my_script = R"(//?
    local a,b,c = ... 
    print(a,b,c)
)";
sol::load_result fx = lua.load(my_script);
if(!fx.valid()) {
    sol::error err = fx;
    std::cerr<<"failed to load string-based script in the program"<<err.what()<<std::endl;
}
fx("your","arguments","here");
```

### 转序列化
不是，这个转序列化也太强了吧。。
```c++
sol::load_result lr = lua.load("a = function (v) print(v) return v end");
c_assert(lr.valid())
sol::protected_function target = lr;
sol::bytecode target_bc = target.dump()
auto result2 = lua2.safe_script(target_bc.as_string_view(), sol::script_pass_on_error);
// check if it was done properly
c_assert(result2.valid());

// check in the second state if it was valid
sol::protected_function pf = lua2["a"];
int v = pf(25557)
//注意这里各种前面的类型，所以我就直接都贴过来了。
```
不过流程只能看懂一部分，但是具体逻辑搞不懂qaq

### 变量的设置和获取
```c++
lua.set()
lua[""] = ...
lua["a"] = lua.create_table_with("v",2)//a = {v = 2}
lua["a_function"] = []() { return 100; };==a_function = function () return 100 end,
std::function<int()> a_std_function = a_function;
```
### 类型
object和其他sol::来检查lua类型  
设置成nullptr或者set成sol::nua_nil  
注意到这里如果是c++类型的userdata/usertype，“则析构函数仅在垃圾收集器认为适合破坏内存时才运行。如果您依赖于将析构函数设置为时运行的析构函数sol::lua_nil，那么您可能犯了一个错误"

### 疑惑
script 起手都是一个R"()"啊，我也没懂，c++11特性？  
即原始字符串，不会对反斜杠做特殊合理，空白和换行符都是字符串的一部分。妙啊。  
看来c++111417迫在眉睫

### 表
表的操作非常方便。  
sol::table 是一个表，支持各种[]的操作  
但是深入要注意安全  
表的声明非常简单

### 函数
可以绑定函数。
不绑定的函数需要传实例化的参数进去。
即两种如下：
```c++
struct some_class{
    int variable = 30;
};
lua.set_function("v2",&some_class::variable,some_class{});
lua["v1"] = &some_class::variable;
v2(254);
v1(sc,212);
```
c++里可以用self来模拟:带来的语法糖（谁模拟谁也不好说ww

多组返回应用std::tuple<...,...,...>建预设类型的表，或者是直接先用类型后tie起来暂时当一体用
（绝了这也是c++11

多个到lua的返回值
```c++
lua["f"] = [](int a,int b,sol::object c){
    // sol::object can be anything here: just pass it through
    return std::make_tuple(a,b,c)
}
```

### c++类中的类
所有不是原始类、string类、函数类、指定的sol类: sol::table,sol::thread,sol::error,sol::object，透明参数类型:sol::varidic_arg,sol::this_state,sol::this_environment，usertype<T>类: sol::usertype
的都被设置为userdata+usertype

来了来了，移动语义、浅拷贝（其实不是浅拷贝，只是说是被lua拥有了，然后会被lua的虚拟机在垃圾回收的时候删掉），正常的等号都会拷贝
std::unique_ptr/std::shared_ptr会受到尊重（


### 引用
```c++
// The following stores a reference, and does not copy/move
// lifetime is same as dog in C++
// (access after it is destroyed is bad)
lua["dog"] = &dog;
// Same as above: respects std::reference_wrapper
lua["dog"] = std::ref(dog);
// These two are identical to above
lua.set( "dog", &dog );~
lua.set( "dog", std::ref( dog ) );
Doge& dog_ref = lua["dog"]; // References Lua memory
Doge* dog_pointer = lua["dog"]; // References Lua memory
Doge dog_copy = lua["dog"]; // Copies, will not affect lua
```

### lua中的c++类
鸽了，要用几个大例子的，例子里到处都是看不懂的c++11语法

### 命名空间
用表在lua里模拟c++的命名空间（注意和本身的环境做一下区分)

...先搞c++11吧
