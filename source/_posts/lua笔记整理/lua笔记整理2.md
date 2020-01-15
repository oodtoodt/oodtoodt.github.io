---
title: lua笔记整理(2)
date: 2020-01-13 15:45:32
tags: 
- lua
- company

categories:
- lua
---

整理一下关于lua的笔记

<!--more-->
---

### day6（元表）

#### 笔记
我惊了 我居然卡了。

将一个函数用作__index元方法，让所有只读表共享同一个元表
首先，用__index就必须要访问nil，所以原表必须是空的，然后访问元表的__index。
那么如何共享同一个元表呢？这感觉就很不现实啊，不同只读表用同一元表访问的时候调用__index，在元素不同的情况下怎么保证访问元素正确呢？然后最后的问题在于，为什么只读表使用index方法？

找了一圈也没找到这种只读的实现，只能理解成题目有问题了。不过也不是一无所获，至少对于这个代理的过程轻车熟路了。

我参考了一下，也许是__newindex，这样共享同一个也说得通，然后每次访问之前set一下__index，可能是想表达这样一个东西吧（
但是如果不set就会去访问别的表hhh
但是面临一个问题，就是必须把原表以某种形式存下来，因为代理里肯定没东西，那么代理因为要index别人跑了拐回来就必须调用这个存下来的数据

算了，不搞了，mdzz


弱智字符串不支持遍历 不支持下标
然后记得string_to_table是有要求的，那就是形式上是符合table格式的！
如
```lua
local string_1 = '{ [name] = "string_1", [color] = "red" }'
local string_2 = '{ [name] = "string_2", [color] = "blue" }'
local string_3 = '{ [name] = "string_3", [color] = "green" }'
local function str2tbl(str)
   return assert((loadstring or load)('return '..str:gsub('%[(.-)]','["%1"]')))()
end
```

另外放弃你那弱智的遍历高速的思想，我求。


t1 = {nil}
t2 = {1,nil,2}
t3 = {1,nil,2,nil}
t4 = {1,nil,2,nil,3}
t5 = {1,nil,2,nil,3,nil}
t6 = {1,nil,2,nil,3,nil,nil}
print(#t1,#t2,#t3,#t4,#t5,#t6)
--[[0   3   1   5    3    5]]

惊了，真实神奇，简直就是switch出来的数据

突然卡壳，不知道对偶的stack怎么写
正常而言，一个对偶接近于一个倒排索引，虽然不是调换key和value位置就是了
```lua
a[key] = value---key[a] = value
```
然后很反直觉的，对于一个双向队列来说，key[a]取出的东西是什么就非常奇怪
如果是值，那么正反向获取不到，如果是位置，那么就必须存两端的位置，然后另外开一个东西记录数据。（其实我不很懂这么两次用表的意义）（在于继承和私有）
最后没开，直接就用的self，对偶扔外面了

 
#### 然后还是写了？实现只读表
可能是有问题的，应该是吧，烂摊子不太想管
```lua
do 
    local mt = {}
    mt.__newindex = function (t,k,v)
        error("attempt to update a read-obly table",2)
    end
    function setVisT(t)
        mt.__index = t.vvvIn
    end
    function readOnly(t)
        local temp = {}
        temp.vvvIn = t
        --setVisT(t)
        setmetatable(temp,mt)
        return temp
    end
    local a = {4,5,6}
    local b = readOnly({1,2,3})
    setVisT(b)
    --a[1] = 2
    print(inspect(a))
    print(b[2])
end
```
#### 代理使得pairs(t)遍历一个文件的所有字节
创建一个以文件为参数，返回为文件的代理的代理表
```lua
do 
    function fileasArray(fn)
        local proxy = {}
        local fr = io.open(fn,"r")
        local alfile = fr:read("a")
        local str2tab = {}
        for i = 1,#alfile do
            str2tab[i] = string.sub(alfile,i,i)
        end
        local mt = {
            __index = str2tab,
            __newindex = function (_,i,ch)
                str2tab[i] = ch
            end,
            __len = function() return #str2tab end,
            __tostring = function ()
                local l = {}
                for _,v in pairs(str2tab) do
                    l[#l + 1] = tostring(v)
                end
                return "{" .. table.concat(l,"") .. "}"
            end,
            __pairs = function()
                local i = 0
                return function()
                    i = i + 1
                    if i > #str2tab then return nil end
                    return i,str2tab[i]
                end
            end, 
        }
        setmetatable(proxy,mt)
        return proxy
    local t = fileasArray(f1)
    local t2 = fileasArray(f1)
    print(inspect(t))
    print(t)
    t[1] = "?"
    t[10] = nil
    print(t)
    print(#t2,#t)
    for _,v in pairs(t) do print(v) end
end
```

### day7（环境）

#### 笔记
把旧环境装入新环境的方式是使用继承
```lua
a = 1
local newgt = {}
setmetatable(newgt,{__index = _G})
_ENV = newgt
print(a)
a = 10
print(a,_G.a)--10       1
```
应该这么讲：之后的赋值都发生在新的表中，与全局环境无关了，但是这跟表的继承相悖啊。为什么改了_G，_ENV不会变？好怪
做了下实验，懂了。因为这里的a = 10直接就给赋值了，所以第二次就没动__index元方法而是跑的_ENV本身的。还是太不熟了。

模式匹配真难啊，匹配所有.然后split写了一中午
先是[%w_]*%.匹配到最后尝试匹配最后一个.
然后想了半天没想出好法子，至少不能写一个类似的匹配语句来做，想到的居然是reverse一下然后再gsub("(.-)%.(.*)$","%1")这么整活，依赖于reverse的复杂度，问题是reverse还不够复杂?新开一个表然后遍历一遍。
何苦呢，怎么不再用find或者gmatch"%."到底?
最后，split有简单实现的啊。[^t]+ t是任意字符不好吗
或者干脆您就原字符串后面加个.，最后去了，不也行吗(gmatch根本不需要去掉，加着就行)

虽然写好了，但是错误处理不知道怎么调



弱引用中，不管是键还是值被回收了整个键值对就木了。

然后这个常量函数工厂给我整蒙了
虽然表中的键是弱引用的，但是表中的值却不是弱引用的，由于值不是弱引用的，所以每个函数都存在以强引用，每个函数指向其对象，于是每个键都存在一强引用
似乎作者也不是很想在这里纠结？就是说类似成环的闭包引用了指回闭包的对象的这种情况。

析构重要的是概念，所以必须先占个位

析构器的对象调用析构器前先清理弱引用表的值，调用后清理键，原因在于析构器可能需要访问带有弱引用键的表来保存对象的属性   


#### 么得代码

### day8（垃圾收集）

#### 笔记
我太菜了（不知道怎么写出一个证明lua需要瞬表的代码
弱引用键、强引用值，加上一些嵌套自环，我整个人都不好了

诶？为什么我这里的输出跟网上的截然不同？别人讲得头头是道，我这根本理解不能？
好的__gc 打错成 __gcd了 我佛了
写一下我的猜测哈：第一次count为0意味着所有的析构器已经调用完成了，这时已经完成的应该是清除了每个元素，但是元素悬挂的元表应该只是被标记并没有被清除，第二次就清除


#### 需要解释的代码：
```lua
do 
    local count = 0
    local mt = {__gc = function() count = count - 1 end}
    local a = {}
    for i = 1, 10000 do 
        count = count + 1
        a[i] = setmetatable({},mt)
    end
    collectgarbage()
    print(collectgarbage("count") * 1024, count)
    a = nil
    collectgarbage()
    print(collectgarbage("count") * 1024, count)
    collectgarbage()
    print(collectgarbage("count") * 1024, count)
end

--[[
604716.0        10000
342576.0        0
22576.0         0
]]--
```

### day9（协程）
#### 笔记

他这里用的这个全排列挺反我的直觉的，因为我的思路就是dfs，每次添加一未使用的数据这样；他的思路是每次交换元素继续对剩下的元素做同样的递归操作，因为会按次序都移回来所以保证了能访问到所有排列。

```lua
function permgen(a,n)
    n = n or #a
    if n <= 1 then
        --coroutine.yield(a)
        printResult(a)
    else
        for i = 1,n do
            a[n],a[i] = a[i],a[n]

            permgen(a,n-1)

            a[n],a[i] = a[i],a[n]
        end
    end
end
```
其实整个思路我最不能明白的部分就是为什么需要用协程的迭代器。
可以离线，那么就可以在中间断开搞协程。这样高的好处是啥？没搞明白。调用上更简单？
但是就实现而言，二者没有区别（指思路），单纯就是好用不需要关心内部吗？那么协程之间会不会发生什么呢？

不过这个传参return的方法值得学习（

新的问题：
```lua
for i = 1,#a do
    local t = i
    return function()
        t = t + 1
        return t
    end
end
```
如果这是个迭代器,会得到正确结果吗？我怀疑不会。
因为迭代器上来就返回了第一个函数然后应该就定死了，upvalue是不会在上层变，而只会在迭代器里改变，要充分理解到这一点。
回到我们实际的情况：
```lua
local t = solve(a,n)
for i = 1,#t do
    local co = coroutine.create(function() permgen(t[i],n) end)
    return function()
        local code,res = coroutine.resume(co)
        return res
    end
end
```
co就已经是初值了，因为它是一个协程，所以它是会执行的自己跑到某个地方然后停下，由迭代器再启动它。
但是不可能有第二个co的值。

ok！顺带还搞明白了迭代器，舒服。


然后就又卡了。写一个基于协程库的行迭代器来读文件。问题在于原用的协程是coroutine.running()，取得是当前的协程，我这里要读文件应该怎么处理？需要对应不同的协程？for循环里还需要指定协程吗？不指定的话，直接使用当前协程不会出问题吗？
整个流程很明确倒是，因为行迭代器无非就是每次读的时候resume，yield，然后可能稍微卡一下传参，但是搞不清楚前面这些我就很憋，不想写。

使用基于协程库来同时运行多个线程。一个简单的场景是多文件一起读写，问题是这里我们数据不够大根本看不出区别，只要没bug应该就是成功的。
不想写，鸽一会。


两个结合在一起我觉得可以搞。
但是问题就是迭代器要求的是直接遍历，总感觉有些冲突，我再读点网上材料。网上没讨论这个的。。我的问题在于多线程在这个问题里到底怎么解释，就是说我是同时遍历所有文件对吧？

那如果，不能迭代器结合多线程，那我写基于协程库的迭代器意义在哪里呢
草26章有讲多线程，回头说回头说...什么鬼设计嘛
**（别想了，回头也什么也说不出来**
 
#### 使用生产者驱动重写生产者消费者示例
```lua
do 
    function consumer()
        while true do 
            local x = receive()
            io.write(x,"\n")            
        end
    end
    function producer()
        while true do 
            local x = io.read()
            send(x)
        end
    end
    function receive()
        local value = coroutine.yield()
        return value
    end
    function send(x)
        coroutine.resume(consumer,x)
    end
    consumer = coroutine.create(consumer)
    print(coroutine.status(consumer))
    coroutine.resume(consumer)
    --producer()
end
--以下是消费者驱动
do 
    function producer()
        while true do 
            local x = io.read()
            send(x)
        end
    end
    function consumer()
        while true do 
            local x = receive()
            io.write(x,"\n")            
        end
    end
    function receive()
        local status,value = coroutine.resume(producer)
        return value
    end
    function send(x)
        coroutine.yield(x)
    end
    producer = coroutine.create(producer)
    print(coroutine.status(producer))
    --consumer()
end

```

#### 编写一个函数输出制定数组的所有组合，使用协程修改为组合的生成器
我用了个二进制的模拟，意外的难写ww
也许有更直接的思路吧，不过我这脑子真转不来了。
~=4，所以里面其实控制位数没写好？应该是啥？#a吧
```lua
--[[
    function permutations(a)
        local co = coroutine.create(function()permagen(a)end)
        return function()
            local code,res = coroutine.resume(co)
            return res
        end
    end

    等同于

    function permutations(a)
        return coroutine.wrap(function()permgen(a) end)
    end
]]

do
    local function solve(a,n)
        print("?")
        local num = 2^(#a)
        local t = {}
        for i = 0,num-1 do 
            local temp = i
            local t_now = {}
            local t_pos = 1
            while(temp ~= 0) do
                t_now[t_pos] = temp % 2
                temp = temp//2
                t_pos = t_pos + 1
            end
            while #t_now ~= 4 do 
                t_now[#t_now + 1] = 0
                --if #t_now < 10 then print(#t_now) end
            end
            local t_now_now = {}
            for i,v in pairs(t_now) 
                do if v == 1 then t_now_now[#t_now_now+1] = a[i] end 
            end
            t_now = nil
            --控制
            if #t_now_now == n then
                t[#t+1] = t_now_now
            end
            --print(inspect(t_now_now),i)
        end
        return t
        --printResult(a,t)
    end
    function permutations(a,n)
        local t = solve(a,n)
        local i = 1
        local len = #t
        local co = coroutine.create(function() permgen(t[i],n) end)

            return function()
                local code,res = coroutine.resume(co)
                if res == nil then 
                    i = i + 1
                    if i > #t then return nil end
                    co = coroutine.create(function() permgen(t[i],n) end)
                    code,res = coroutine.resume(co)
                end
                return res
            end
    end
    function permgen(a,n)
        n = n or #a
        if n <= 1 then
            coroutine.yield(a)
            --printResult(a)
        else
            for i = 1,n do
                a[n],a[i] = a[i],a[n]

                permgen(a,n-1)

                a[n],a[i] = a[i],a[n]
            end
        end
    end
    function printResult(a)
        for i = 1, #a do 
            --if t[i] == 1 then io.write(a[t[i]]," ") end
            io.write(a[i]," ")
        end
        io.write("\n")
    end
    for p in permutations({1,2,3,4},3) do
        --sprintResult(p)
    end
end
```

### day10（反射）
老实说我没什么印象了
hook还有一点印象ww

#### 笔记

改进了getvarvalue使之可以处理协程，见下，但是必须要传我现在这个协程进去。不过想想也是必须的不是嘛（。
改值这居然有坑等着我，就当修改global变量的时候取_ENV之后居然是直接出上值的。感觉我是强行修的。不过似乎，也许只有这一种情况吧。

第三题又没看懂：是要我做一个随时调用每次调用都会显示当前函数内所有可见变量还是说参数里带函数名的去找?
有点类似离线还是在线的意思，就我当前这体感来说，debug都是离线的。

mgj这几个题读起来都费劲。
第四题：
「该函数在调用debug.debug函数的词法定界中运行指定的命令」
what the fuck
在debug.debug的词法定界里跑指定的函数。
根据提示的话，这里应该是用__index元方法=getvarvalue得出的某个upvalue表这样，简单的复习了一下词法定界，感觉没什么问题。upvalue的upvalue其实在upvalue里就是普通变量而已（针对我的某个疑惑）


好挫败啊，昨天和今天的基本上都没弄懂。
昨天的垃圾分类器马马虎虎根本不知所以，今天协程反射重拳出击我人都打傻了。

注意到这个25.3是 返回 一个表，那么应当是在线的了。

我一开始想写的是直接扫描到我这个函数处在的位置的所有可用（见）变量，但是好像做不到。我不知道怎么获取到我调用这个的位置。比如traceback是显示栈范围内的变量，原getvalue是指定变量名搜索，我们现在的需求是找到当前范围内的可见变量。一个想法是传进我当前函数名。但是这样肯定不够智能。一个可选的操作是用钩子？调用这个函数的时候返回，而调用这个函数之前一直记录直到分成两种情况：当前函数在这个函数里，不在（在另一个外层函数里），不过上下层其实分析不出来，总不能每次都来一个getlocal跑个level出来吧，这么麻烦不如传函数名呢还。
放弃啦（
就传函数名，并且无视掉重名的各种local情况（无非就是表中表，先记录下所有的位置，然后跑循环。懒得弄了）

我tm惊奇的发现原来的getvalue tmd本来就只能获取到当前位置的变量。我傻了。
问题出在这个debug.getinfo(level,"f").func

认认真真的读了一下这个getinfo的level的意义：当前getinfo为0，调用getinfo的函数A的层次为1，调用A函数的函数的层次是2，getlocal同理，所以默认从2的local找起就是此理。
这里要纠正一个思维上的误区：getinfo找到所有当前层次的函数，不，只会找到调用getinfo当前这个函数栈内的层层函数。
懂了吗（？
我佛了，当时是怎么写出协程版本的getvalue的。。。。估计就照抄来的：为什么level-1呢，就是要直接获取第一层的函数：因为这时候有协程参数，会直接进协程这个函数体内（如果有？必须有吧？）

emmm，我有很多发现。
比如如果我放到迭代器里，会多出好几个层次的表，而且都是loc的，loc的loc又包含一个新的表，而且有迭代专属的几个变量，是很好的分析材料（。
单独放的时候，不知道为什么打印数据用的inspect会被加进表里。还是放进的loc表，说明是个隐藏的局部变量？藏在每个层次里？有待深究，不过意义不大。

upvalue里有一个_ENV，boom

大概明了的一点就是upvalue只在使用之后才会成为upvalue，然后就算你放到了一个函数里面，调用的地方还是会使得整个值发生变化
具体来说就是：
```lua
local t = 11
function()
local u = 111
t = t + 1
local ans = getvarvalue()
return ans

---> loc:n=u,v=111;upv:n=t,v=12;

local t = 11
function qq()
local u = 111
t = t +1
return getvarvalue()

ans = qq()

--->loc:n = t,v = 12;n = "it"（这是另一个local函数),v = function 3; n = "qq,v = function 4；upv:nil


local t = 11
function()
local u = 111
t = t + 1
print(inspect(getvarvalue()))

---> loc: n = u, v = 111;n = "(*temporary)", v = function 1;n = "(*temporary)", v = <1>{KEY = inspect.KEY......}
```
这里可能忽略了一些引号，但不是重点。重点在于三种使用在逻辑上我认为是等效的，但其实不然。

其实作为迭代器里的return会出现更加难以捉摸的结果，不过我已经不想动了。

函数还能更改的地方在于不断up level，找到up level之后的值扔进新的表里做层次递进，这样比单纯的用upvalue好多了（体验过这乱七八糟不靠谱之后）
不想整活了，留个坑，以后填。


26章居然就是网上一篇资料的原文出处，我傻了。事到如今又看了点资料，感觉我对协程是有理解了，但是我对怎么写还很模糊。
另一个感慨就是要多学点底层的东西。
回过头来看这个24.5，能否基于它来跑多线程?可以。修改？把running给夜改了，改成传参的方式可能好一点，方便调用。

想跑一下26的代码都么得，难过。环境啊qaq


盲猜一下24.4的代码：
```lua
function getline()
    local co = coroutine.running()
    return function()
        local callback = function(l) coroutine.resume(co,l)end 
        lib.readline(stream,callback)
        return coroutine.yield(co)
    end 
end

for line in getline(inp) do 
    ...
end
```

这难度跨越真就不讲道理啊。
编写一个允许脚本限制其lua状态能够使用的总内存大小的库。设置自己的内存分配函数，检查总量。
初步考虑是newstate调用自己写的分配函数。具体还要多构思一些。

#### 修改getvarvalue
```lua
do --first try no need to see
    function getsetvalue(name,val,level,isenv)
        local value 
        local found = false
        level = (level or 1) + 1
        local pre_value
        for i = 1,math.huge do 
            local n,v = debug.getlocal(level,i)
            if not n then break end
            if n == name then
                found = true
                pre_value = v
                debug.setlocal(level,i,val)
                _,value = debug.getlocal(level,i)
            end
        end
        if found then return "local", pre_value,value end


        local func = debug.getinfo(level,"f").func
        for i = 1,math.huge do
            local n,v = debug.getupvalue(func,i)
            if not n then break end
            if n == name then 
                if(n == "_ENV") then return nil,nil,v end
                debug.setupvalue(func,i,val)
                local temp = v
                _,v = debug.getupvalue(func,i)
                return "upvalue",temp,v 
            end
        end
        
        if isenv then return "noenv" end

        local _,_,env = getsetvalue("_ENV",val,level,true)
        if env then 
            local value = env[name]
            env[name] = val
            return "global",value,env[name]
        else 
            return "no env" 
        end
    end
    local a = 11
    b = 12
    print(getsetvalue("a",14))
    print(getsetvalue("b","nzk"))
end

do 
    function getvarvalue(name,level,isenv,co)
        local value 
        local found = false
        level = (level or 1) + 1

        if co then
            level = level - 1
            for i = 1,math.huge do 
                local n,v = debug.getlocal(co,level,i)
                if not n then break end
                if n == name then
                    value = v
                    found = true
                end
            end
            if found then return "local", value end


            local func = debug.getinfo(co,level,"f").func
            for i = 1,math.huge do
                local n,v = debug.getupvalue(func,i)
                if not n then break end
                if n == name then return "upvalue" ,v end
            end
            
            if isenv then return "noenv" end

            local _,env = getvarvalue("_ENV",level,true,co)
            if env then 
                return "global",env[name]
            else 
                return "no env" 
            end
        else 
            for i = 1,math.huge do 
                local n,v = debug.getlocal(level,i)
                --print("---",n,v,isenv,"---")
                if not n then break end
                if n == name then
                    value = v
                    found = true
                end
            end
            if found then return "local", value end


            local func = debug.getinfo(level,"f").func
            print(func)
            for i = 1,math.huge do
                local n,v = debug.getupvalue(func,i)
                --print("---",n,v,isenv,"---")
                if not n then break end
                if n == name then return "upvalue" ,v end
            end
            
            if isenv then return "noenv" end

            local _,env = getvarvalue("_ENV",level,true)
            if env then 
                return "global",env[name]
            else 
                return "no env" 
            end
        end
    end    
    co = coroutine.create(function() local x = 10 fuck = "fuck you you little dummy" coroutine.yield() error("some error") end)
    coroutine.resume(co)
    print(getvarvalue("x",nil,nil,co))
    print(getvarvalue("fuck",nil,nil,co))
    local function nowwhat(a) return a + 1 end
    a = function ()
        local t = 1 
        return function() 
            local e = 11 
            print(getvarvalue("t"))
            return t + 1 
        end 
    end
    f = a()
    f()
    --debug.debug()
end
```

#### 获取当前环境中可见变量的getvarvalue
```lua
package.path = package.path .. ";./?.lua"
local inspect = require("inspect")

do 
    a = 1
    function getvarvalue()
        local level = 2
        local valtab = {}
        valtab.loc = {}
        valtab.upv = {}
        valtab.gal = {}
        for i = 1,math.huge do 
            local n,v = debug.getlocal(level,i)
            --print("---",n,v,isenv,"---")
            if not n then break end
            valtab.loc[#valtab.loc+1] = {}
            valtab.loc[#valtab.loc].n = n
            valtab.loc[#valtab.loc].v = v
        end


        local func = debug.getinfo(level,"f").func
        local funcn = debug.getinfo(level,"n").name
        print(funcn)
        for i = 1,math.huge do
            local n,v = debug.getupvalue(func,i)
            
            if n ~= "_ENV" then
            --print("---",n,v,isenv,"---")
            if not n then break end
            valtab.upv[#valtab.upv+1] = {}
            valtab.upv[#valtab.upv].n = n
            valtab.upv[#valtab.upv].v = v
            end
        end
        
        --[[local ENV = _ENV
        for i,v in pairs(ENV) do 
            valtab.gal[#valtab.gal+1] = {}
            valtab.gal[#valtab.gal].n = i
            valtab.gal[#valtab.gal].v = v
        end
        --]]
        return valtab
    end
    local t = 11
    local function it()
        local u = 111
        function aa()
            u = u + 1
            a = a + 1
            if u >= 120 then return nil end
            return getvarvalue()
        end
        return aa
    end 
    local function qq()
        local u = 111
        t = t + 1
        local ans = getvarvalue()
        return ans
    end
    for u in it() do 
        --print(inspect(u))
    end
    ans = qq()
    f = it()
    print(inspect(ans))
    --ans = f()
end

```
后面还有关于接口调用一类的，不过...下篇见。