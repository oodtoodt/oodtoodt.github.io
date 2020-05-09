---
title: sol2笔记-2
date: 2020-05-08 11:24:06
tags:
---

时隔近半年，重回sol2学习
<!--more-->

---

## 前言
这次其实就是带着代码读一遍而不是简单的过一遍了。
主要是我能看懂这些lambda不至于慌神迷乱了。
## getting-started
起手`sol::state lua;lua.open_libraries(sol::lib::base)`没问题。

## 读写
```c++
int is_defaulted = lua[".."][".."].getor(24);
auto bark = lua["config"]["bark"];
if(bark.valid()) {} 
sol::optional<bool> is_a_boolean = lua["config"]["brak"];
if(is_a_boolean) {/*the value is a boolean*/}
```
写
写的时候注意嵌套
```c++
lua["bark"] = 50;

lua["some_table"] = lua.create_table_with(
    "key0", 24,
    "key1", 25,
    lua["bark"], "the key is 50 and this string is its value!");
lua.script(R"(
    
print(some_table[50])
print(some_table["key0"])
print(some_table["key1"])

-- a lua comment: access a global in a lua script with the _G table

print(_G["bark"])

)");
/*
the key is 50 and this string is its value!
24
25
50
*/
```
同时写的时候注意不要进行太深的写，容易造成在不存在的table里写东西，这时候可以用一些lazy读写，最后查看其valid()。
如果一个值为nil(sol::lua_nil)，那么`sol::optional<int>`将在if中变成false。

## 函数
### 注册函数
三种方法：
```c++
std::string my_func(int a, std::string b) {
    return b + std::string("D", a);
}
int main(){
    sol::state lua;
    lua["a"] = my_func;
    lua.set("b",my_func);
    lua.set_function("c",my_func);
    lua.script("some_str = a(1, 'Da')");
    std::string some_str = lua["some_str"];
}
```
```c++
struct my_class {
    int a = 0;
    my_class(int x) : a(x) {

    }
    int func() {
            ++a; // increment a by 1
            return a;
    }
};
lua.set_function("my_class_func", &my_class::func, my_class());
lua.set_function("my_class_func_2", &my_class::func);
lua.set("obj", my_class(24));
lua.script(R"(
    first_value = my_class_func()
    second_value = my_class_func()
)");
// first_value == 1
// second_value == 2
lua.script(R"(
    third_value = my_class_func_2(obj)
    fourth_value = my_class_func_2(obj)
)");
// first_value == 25
// second_value == 26
```
模板是很难注册的，因为它是编译期内容，如果你需要，指定所有的参数类型才可以绑定和使用
可以用sol::overload重载函数
lambda会有很多奇怪的问题。记住，用my_table.set_function注册所有的lambda，我相信这里set_function和所有都是关键的点。然后没有指定返回类型的lambda会衰退(decay，std::decay就是对类型退化的意思，这里应该同理，去const reference)返回值，要显式捕获或返回引用，根据需要使用decltype(auto)或直接指定返回类型
问题的原因也很有趣，这里不展开了。
>Welcome to non-static-reflection hell

### 函数与参数传递
这里非常的复杂，至少我大部分没有看懂。留个心眼吧。
+ 所有参数都会被转发。意味着不会发生复制、移动，除非明确由用户或接收函数完成。
+ sol::table,sol::object,sol::userdata and friends(?)复制代价很低，并且简单的视为值，如果您不希望使用copy，那就要用const type&或者type&，注意用引用很危险，因为lua默认传引用，值很容易被更改。
+ 函数绑定到lua时，指针参数都用作`T*`
+ 避免通过引用或值使用特殊的unique_usertype参数，因为许多类型只能移动，而lua没有移动的概念，通过引用是非常危险的。
+ 函数有对nil/nullptr的期望就可以用`T*`,没有就用(const T&/T&)【注:没看懂】
+ 尽量不要用`const char*`，除非你对Lua Stack了如指掌

### 从lua获取函数
使用sol::function或使用sol::protected_function这种更高级的包装器
前者很简单，如果不用的话其实auto也可以，或者干脆做右值，都行
后者可以绑定一个error_handler变量，捕获错误并处理他们
```c++
int main () {
        sol::state lua;

        lua.script(R"(
                function handler (message)
                        return "Handled this message: " .. message
                end

                function f (a)
                        if a < 0 then
                                error("negative number detected")
                        end
                        return a + 5
                end
        )");

        sol::protected_function f = lua["f"];
        f.error_handler = lua["handler"];

        sol::protected_function_result result = f(-500);
        if (result.valid()) {
                // Call succeeded
                // Which would be not called here
                int x = result;
        }
        else {
                // Call failed
                sol::error err = result;
                std::string what = err.what();
                // 'what' Should read
                // "Handled this message: negative number detected"
        }
}
```

```c++
#define SOL_ALL_SAFETIES_ON 1
#include <sol/sol.hpp>

#include <iostream>
#include "assert.hpp"

// Uses some of the fancier bits of sol3, including the "transparent argument",
// sol::this_state, which gets the current state and does not increment
// function arguments
sol::object fancy_func(sol::object a, sol::object b, sol::this_state s) {
	sol::state_view lua(s);
	if (a.is<int>() && b.is<int>()) {
		return sol::object(lua, sol::in_place, a.as<int>() + b.as<int>());
	}
	else if (a.is<bool>()) {
		bool do_triple = a.as<bool>();
		return sol::object(lua, sol::in_place_type<double>, b.as<double>() * (do_triple ? 3 : 1));
	}
	// Can also use make_object
	return sol::make_object(lua, sol::lua_nil);
}

int main() {
	sol::state lua;

	lua["f"] = fancy_func;

	int result = lua["f"](1, 2);
	// result == 3
	c_assert(result == 3);
	double result2 = lua["f"](false, 2.5);
	// result2 == 2.5
	c_assert(result2 == 2.5);

	// call in Lua, get result
	// notice we only need 2 arguments here, not 3 (sol::this_state is transparent)
	lua.script("result3 = f(true, 5.5)");
	double result3 = lua["result3"];
	// result3 == 16.5
	c_assert(result3 == 16.5);
	
	std::cout << "=== any_return ===" << std::endl;
	std::cout << "result : " << result << std::endl;
	std::cout << "result2: " << result2 << std::endl;
	std::cout << "result3: " << result3 << std::endl;
	std::cout << std::endl;

	return 0;
}
```
这里的sol::this_state,sol::state_view值得玩味，因为文档里没有提他们的用途（到这为止），盲猜就是绑定了环境

## lua中的c++
使用usertype或udt很简单。如果不调用任何成员变量和函数，直接传递即可。
如果要在lua的用户类型上使用变量和函数，就需要注册：可用`new_usertype`绑定方法和变量，这些方法都在table和state(_view)上，但我们只在state上使用它
另外new_usertype允许一些设置选项，详见文档。
```c++
if (way_1) {
        lua.new_usertype<ship>( "ship", // the name of the class, as you want it to be used in lua
                // List the member functions you wish to bind:
                // "name_of_item", &class_name::function_or_variable
                "shoot", &ship::shoot,
                "hurt", &ship::hurt,
                // bind variable types, too
                "life", &ship::life,
                // names in lua don't have to be the same as C++,
                // but it probably helps if they're kept the same,
                // here we change it just to show its possible
                "bullet_count", &ship::bullets
        );
}
else {
        // set usertype explicitly, with the given name
        sol::usertype<ship> usertype_table = lua.new_usertype<ship>( "ship");
        usertype_table["shoot"] = &ship::shoot;
        usertype_table["hurt"] = &ship::hurt;
        usertype_table["life"] = &ship::life;
        usertype_table["bullet_count"] = &ship::bullets;
}
```

## 所有权
在销毁sol::state之前，必须销毁所有对象，否则悬挂引用Lua State会造成可怕的爆炸
在这里，sol::object在状态超出范围前，必须清除所有源自sol::reference和sol::object的类型
这段代码很熟悉，但是不记得是在哪看过了
```c++
sol::state lua;
	lua.open_libraries(sol::lib::base);

	lua.script(R"(
	obj = "please don't let me die";
	)");

	sol::object keep_alive = lua["obj"];
	lua.script(R"(
	obj = nil;
	function say(msg)
		print(msg)
	end
	)");

	lua.collect_garbage();

	lua["say"](lua["obj"]);
	// still accessible here and still alive in Lua
	// even though the name was cleared
	std::string message = keep_alive.as<std::string>();
	std::cout << message << std::endl;
```

### 指针
>sol不会拥有原始指针的所有权：原始指针不拥有任何东西。sol不会删除原始指针，因为它们不（也不应该拥有）任何东西：

```c++
#define SOL_ALL_SAFETIES_ON 1
#include <sol/sol.hpp>

struct my_type {
	void stuff() {
	}
};

int main() {

	sol::state lua;
	// AAAHHH BAD
	// dangling pointer!
	lua["my_func"] = []() -> my_type* { return new my_type(); };

	// AAAHHH!
	lua.set("something", new my_type());

	// AAAAAAHHH!!!
	lua["something_else"] = new my_type();
	return 0;
}
```
这里就是之前提到过的，lambda一定要指定返回值，其他的同理：
```c++
#define SOL_ALL_SAFETIES_ON 1
#include <sol/sol.hpp>

struct my_type {
	void stuff() {
	}
};

int main() {

	sol::state lua;
	// :ok:
	lua["my_func0"] = []() -> std::unique_ptr<my_type> { return std::make_unique<my_type>(); };

	// :ok:
	lua["my_func1"] = []() -> std::shared_ptr<my_type> { return std::make_shared<my_type>(); };

	// :ok:
	lua["my_func2"] = []() -> my_type { return my_type(); };

	// :ok:
	lua.set("something", std::unique_ptr<my_type>(new my_type()));

	std::shared_ptr<my_type> my_shared = std::make_shared<my_type>();
	// :ok:
	lua.set("something_else", my_shared);

	// :ok:
	auto my_unique = std::make_unique<my_type>();
	lua["other_thing"] = std::move(my_unique);

	return 0;
}
```
尽量使用std::nullptr_t或sol::lua_nil
```c++
#define SOL_ALL_SAFETIES_ON 1
#include <sol/sol.hpp>

struct my_type {
	void stuff() {
	}
};

int main() {

	sol::state lua;
	// THIS IS STILL BAD DON'T DO IT AAAHHH BAD
	// return a unique_ptr that's empty instead
	// or be explicit!
	lua["my_func6"] = []() -> my_type* { return nullptr; };

	// :ok:
	lua["my_func7"] = []() -> std::nullptr_t { return nullptr; };

	// :ok:
	lua["my_func8"] = []() -> std::unique_ptr<my_type> {
		// default-constructs as a nullptr,
		// gets pushed as nil to Lua
		return std::unique_ptr<my_type>();
		// same happens for std::shared_ptr
	};

	// Acceptable, it will set 'something' to nil
	// (and delete it on next GC if there's no more references)
	lua.set("something", nullptr);

	// Also fine
	lua["something_else"] = nullptr;

	return 0;
}
```

使用（protected_）function_result，load_result（尤其是多个加载/函数会导致一个C ++函数），stack_reference和类似基于堆栈的事物时，请务必小心，以他们为返回值前三思。
proxy依赖于Lua堆栈，从c++函数中返回它们完全不安全，而且没有什么好的解决办法。

### 代理
```c++
#define SOL_ALL_SAFETIES_ON 1
#include <sol/sol.hpp>

#include "assert.hpp"

#include <iostream>

int main () {

	const auto& code = R"(
	bark = { 
		woof = {
			[2] = "arf!" 
		} 
	}
	)";

	sol::state lua;
	lua.open_libraries(sol::lib::base);
	lua.script(code);

	// produces table_proxy, implicitly converts to std::string, quietly destroys table_proxy
	std::string arf_string = lua["bark"]["woof"][2];

	// lazy-evaluation of tables
	auto x = lua["bark"];
	auto y = x["woof"];
	auto z = y[2];

	// retrivies value inside of lua table above
	std::string value = z;
	c_assert(value == "arf!");

	// Can change the value later...
	z = 20;

	// Yay, lazy-evaluation!
	int changed_value = z; // now it's 20!
	c_assert(changed_value == 20);
	lua.script("assert(bark.woof[2] == 20)");

	lua["a_new_value"] = 24;
	lua["chase_tail"] = [](int chasing) { 
		int r = 2;
		for (int i = 0; i < chasing; ++i) {
			r *= r;
		}
		return r;
	};

	lua.script("assert(a_new_value == 24)");
	lua.script("assert(chase_tail(2) == 16)");

	return 0;
}

```
看起来没有proxy的内容，其实auto的xyz都是`sol::table_proxy<...>`代理类。它们惰性求值，并且被当做引用来使用。当然我们不建议使用proxy在类和类之间或函数和函数之间用惰性求值
更深的东西请查阅文档和api，这里不展开。