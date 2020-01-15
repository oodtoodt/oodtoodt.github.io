---
title: lua笔记整理(1)
date: 2020-01-13 10:54:01
tags:
- lua
- company

categories:
- lua
---

整理一下关于lua的笔记

<!--more-->
---

### day1
#### 笔记
type返回的是string
p就是 * 2^n,e就是 * 10^n
基本类型是值传递，只有表是引用传递 草我人傻了
字符串是常量（？)，且字符串无法迭代 草我人傻了

我有一种寻最优解的习惯，是c的坏习惯，在脚本中其实不用那么在意的。【【都tm脚本了还那么在意效率干啥

非常神秘的卡在读\n上
为什么遇到\n要后退两格才能正确处理?是\r\n的问题吗？搞了将近一个小时没搞明白

### day2，3（string）
#### 笔记
这个函数式编程很神奇
但是我数学不咋地，没看懂这个画蛾眉月的过程：先画圆，求一个增量平移，然后求一个差集，就是月亮了，最后从-1,1映射到n,m文件里
过程好理解，扔进公式里就真不懂了233

这个前置模式我是真没看懂
我唯一能够想到的解释就是在两个%f[char-set]之间的匹配是严格的

这个嵌套递归格式转换器有东西

string.find可以有第三个返回值，但是必须加括号匹配（即返回的其实是捕获、其他的应该同理。
真想要那个找到的字符串不如直接match
[]自主字符集，()捕获

注意都是常量嗯

对于"\0\1"，这是两个字符，不需要单独的\判断处理。对于"\"可以
所以对于书里那个"\\\\(.)"的匹配深表怀疑，不知道怎么回事。
it seems that code and decode have no use
有没有都一样 可能以前的时候可以当两个字符获取？现在就一个了。
（注，最后一段是我代码写错了产生了误解，不用看）
指对于\\\\(.)这种带斜杠转义（比如后面跟引号）的字符串进行匹配时先code再decode可以进行预处理以方便转义引号。
```lua
local kk = [[follows a typical string:"This is \"great\"!".]]
kk = string.gsub(kk,'".-"',string.upper)--是问题的
```

[]中逻辑是或，指集合里面任意皆可，()的匹配就是完全匹配

#### 预置的字符分类
+ .  -->  任意字符 &nbsp;
+ %a -->  字母 &nbsp;
+ %c -->  控制字符 &nbsp;
+ %d -->  数字 &nbsp;
+ %g -->  除空格外的可打印字符 &nbsp;
+ %l -->  小写字母 &nbsp;
+ %p -->  标点符号 &nbsp;
+ %s -->  空白字符 &nbsp;
+ %u -->  大写字母 &nbsp;
+ %w -->  字母和数字! &nbsp;
+ %x -->  十六进制数字 &nbsp;

---

+ \+ --> 一次或多次
+ \* --> 零次或多次
+ \- --> 零次或多次（最小匹配）
+ ? --> 零次或一次

---
#### %f的蜜汁使用
前置模式%f，我的理解就是匹配前一个字符在[char-set]外，后一个字符在[char-set]内的中间空位（前置）。
```lua
local s = "the anthem is the theme aathe what   tx  the  "
print((string.gsub(s,"%f[%w]the%f[%W]","one")))
--one anthem is one theme aathe what   tx  one
```

#### split分割字符串(string,pattern)
```lua
function split(str,par)
    local a = {}
   --[[
       for s in string.gmatch(str,".-%f["..par.."]") do--(.-%f[ ])
       table.insert(a,s)
    end
    ]]
    for s in string.gmatch(str,"[^"..par.."]+") do--[^ ]$
        table.insert(a,s)
    end
    for i,v in pairs(a) do
        print(i,v)
    end
end
```

#### transliterate(string,table)将表按a = b修正字符串
牵扯到转义字符，注意使用。
就跟lua的转义符是反斜杠一样，%是转义字符，注意他们。
```lua
function transliterate(str,tab)
    for a,b in pairs(tab) do
        a = string.gsub(a,"(%W)","%%%1")
        b = string.gsub(b,"%%","%%%%")
        print(a,b)
        if(b == false) then 
            str = string.gsub(str,a,"")
        else
            str = string.gsub(str,a,b)
        end
    end
    print(str)
end
transliterate("hello world fuck you !!!!",{["o"] = "%1",["!"] = "h",[" "] = 101})
--hell%1101w%1rld101fuck101y%1u101hhhh
transliterate("hello world fuck you !!!!",{["o"] = "1",["l"] = "h"})
--hehh1 w1rhd fuck y1u !!!!
```

### day4（load和序列化）
其实day3悄咪咪写了个dijkstra，但是实在是太丑陋了所以...只是无限加深了一个印象：table是引用，普通值都是拷贝。
另外呢，我的dijkstra也写得蛮有问题，照抄都不会的（捂脸），太菜了

#### 笔记
就单纯的这个序列化而言，简单的键是没有问题的，但是如果键是table，那么可能要重新设计整个函数，尤其是递归的逻辑我还是不太搞得清楚

然而$表[表] = 表$的构造器很反直觉啊，完全不知道是要怎么

学会了f5默认文件2333 是json哒

然后大概把这个东西写出来了，最后那题真不想写了。带循环的这套逻辑，回头补吧。

写了个stringrep_n 的load，对load感到困惑。
成了，经过几个小时的实验，证明了一件事:不要在load里面写出function-end，不会认的。或者说，load会自动添加一个function-end的壳子，而且是必然添加。
写到这里劳资终于明白了为什么套一层不会出结果了，因为这样就成了闭包的非匿名形式，没有最后的return，这个函数就只能是个自闭的函数。
我佛了。
不过还好，弄得清清楚楚明明白白也是好事。

#### 很怪的serialize
```lua
do
    local num = 0
    function serialize(o,flag)
        local t = type(o)
        local temp
        flag = flag or 0
        if t == "number" or t == "string" or t == "boolean" or t == "nil" then
            return string.format("%q",o)
        elseif t == "table" then
            num = num + 1
            temp = string.rep(" ",num*4)
            if flag == 0 then io.write("{\n")
            elseif flag == 1 then 
              --  num = num + 1
                --temp = string.rep(" ",num*4)
                local te = string.rep(" ",num*4-8)--?
                io.write("" ..te.. "{\n")
            end
            for k,v in pairs(o) do
                if type(k) == "table" then 
                    io.write(temp.." ")
                    serialize(k,1)
                    io.write(" = ")
                    --num = num - 1
                elseif type(k) == "string" then 
                    io.write(temp.." ",string.format(" [%s] = ",serialize(k)))
                else
                    io.write(temp.." ", k, " = ")
                end
                io.write(serialize(v))

                io.write(",\n")
            end
            num = num-1
            temp = string.rep(" ",num*4)
            if temp ~= "" then temp = temp.." " end
            io.write(temp.."}")
            
        else
            error("cannot serialize a "..type(o))
        end
    end

    local uu = {}
    local u = {c = "?",d = "e"}
    uu[u] = {a = "1",b = "2"}
    --serialize({a = 12, b = "Lua", key = 'another "one"', uu})

    print(type({a=12}))
end
```

#### stringrep_n的正确版本
```lua
do
    local function stringrep_n(n) 
        local x = "";
        local u = ""
        u = u.."local x = \"\";\n\tlocal s = ...;\n\t"
        while n ~= 0 do
            if n % 2 == 1 then 
                --x = x .. s;
                u = u .. "x = x .. s;\n\t"
            else
                --s = s .. s;
            end
            n = n // 2;
            if n == 0 then break end
            u = u .. "s = s .. s;\n\t"
        end
        u = u.."return x;\n\t"
        return u;
    end
    print(stringrep_n(7))
    local stringrep_7 = load(stringrep_n(7))
    local s = stringrep_7("fku\n")
    print(s)
end

--例子就看这个好了
--load的捕捉是靠...的
do
    local a = "function Add(a,b) return a+b end return Add(...)"
    local f = assert(load(a))
    local F = function (aa,bb) return aa+bb end
    print(F(1,1))
    print(f(2,3))
end
```

### day5（迭代器）

#### 笔记
看起来，这个for无状态循环迭代非常严格，至少应当把控制变量当成一个upvalue而且必须作为返回值否则可能获取不到新值
无状态迭代器： 
function (s)
    return next,s,nil--调用next(s,nil)，nil是控制变量初值，s是不可变值
end

真正的迭代器：就是写个大函数，参数用函数

#### 无状态迭代器
```lua
do
    function Fromto(endtable,now)
        local step_len = endtable.step_len
        local Endpos = endtable.en
        if now >= Endpos then return end
        now = now + step_len
        return now
    end
    function fromto(star,endpos,step_len)
        local endtable = {}
        endtable.en = endpos
        endtable.step_len = step_len
        return Fromto,endtable,star-step_len
    end
    local n = -20;local m = 0;local step_len = 2;
    for i in fromto(n,m,step_len) do 
        print(i)
        --print(Q[i])
    end
end
```
#### 遍历指定集合的所有子集
```lua
do
    function substr(str)
        local s = {}
        local i = 1
        local j = 0
        return function()
            j = j + 1
            if j > #str then
                j = i+1
                i = i + 1
            end
            if j > #str then return nil end
            return string.sub(str,i,j)
        end
    end
    local s = "nzk"
    for v in substr(s) do
        print(v)
    end
end
```



有点长了，我们下篇见。