---
title: org-mode学习笔记_1

date: 2018-02-26 13:02:47

tags:
- Org

categories:
- Emacs

---

gtd相关的一些内容
<!--more-->

这个笔记主要是我看一部分就忘一部分很头疼……

理一个纲出来：
todo状态——包括优先级啊，复选啊，进度啊，多状态啊
标签
属性（属性是带值的tag，或者实现数据库，暂时先不学了，我本来是想这么说的，结果看了一下column视图实在是很好的东西啊。。）
时间戳
capture，也是我一直在用的一个吧算是。
杂食，比如稀疏树一类的知识

## 主食...
#### 大纲之间跳
C-c C-n/p 下/上一个标题
C-c C-f/b 下/上一个同级标题
C-c C-u 上层标题
####
C-x n s/w 视角放大／缩回到某个大纲。



## 杂食

#### 注释
[fn:name] 注释，name是唯一的标签
C-c C-x f 当前创建注释
C-c C-c 对跳
#### 表格
C-c | 选定域转换为表格
别的不管
#### sparse树（稀疏树）
C-c / 特定的sparse树
#### 列表
无序列表 - +
有序后接句号或右括
#### 链接
不会用。可以先用注释代替跳转。。。

## todo状态
C-c C-t
定义各种状态，去init-org.el里改
#### 优先级
C-c , 设置
S-up ／ S-down
#### 子任务
在父任务上[/]或者[%]并C-c C-c
如果要使父任务能够继承子任务的任务情况，需要:COOKIE_DATA: recursive的属性，记得直接是在父标题下面加的

子事项以[]开头，则被当成一个checkbox，然而我无论如何也调用不出来，反而耗费了大量时间=-=按下不表

## Tags
#### 为标题分配tag
C-c C-q 输入新的tag。
另外设置
这句+TAGS: { @work(w)  @home(h)  @tennisclub(t) }  laptop(l)  pc(p)和下面的设置相同（就是lisp翻译）
(setq org-tag-alist '((:startgroup . nil)
,                      ("@work" . ?w) ("@home" . ?h)
,                      ("@tennisclub" . ?t)
,                      (:endgroup . nil)
,                      ("laptop" . ?l) ("pc" . ?p)))
#### 搜索tag
C-c / m 。。。记得吗！sparse tree。
C-c a m agenda中搜索tag
C-c a M agenda中搜索tag但只带todo（并强制子树匹配

## 属性和column
#### 属性声明
：PROPERTIES： 特殊抽屉，每个属性一行，键在前，值在后。记得全带:content:
#### 特殊属性
。。。太多啦 一般就是todo，tags，priority，deadline，scheduled
#### 搜索属性
C-c / m 所有匹配tag搜索的标题
C-c a m agenda搜索所有匹配tag/属性搜索的标题
C-c / p 根据属性搜索匹配
#### 属性继承
属性默认具有继承性，然而默认不打开:( 
#### Column视图
column定义式。。按下
C-c C-x C-c
r或者g 刷新
q 推出
e 编辑值
v 完整的属性值
a 编辑该属性的可选值
#### 捕捉column视图
啧，没用。

## 时间戳
#### 创建
C-c . 插入对应的时间戳
C-c !  无效的时间戳（即不会影响agenda上的显示）
C-c C-c 更正时间戳（星期数
C-u C-c .／C-u C-c !
C-c C-y 根据时间段计算出时间跨度显示在minibuffer上
#### 规划
deadline <.... +1m -5d>每一个月重复一次，表示提前5天发出警示
++指令：比如说更换电池，只需要你在最后一次完成任务的时间点之后开始计算重复
C-c C-d 在标题的下一行插入deadline关键字
C-c C-s 插入scheduled
C-c / d 超期的或快要超期的任务sparse tree
C-c C-x C-k 标记任务为agenda action的目标
#### 计时——Clock
emmm，挺麻烦的，需要一个started状态才能计时
C-c C-x C-i 开始一个计时
C-c C-x C-o 结束
C-c C-x C-q 取消
C-c C-x C-x 在最后一次开计时的地方开计时
C-c C-x C-e 更新效用评估信息 大概的话就是[0:33/0:00]后面那个
##### 计时报告——dynamic block
C-c C-x C-r 建一个详细的表
（C-u）C-c C-x C-u 更新光标所在表（所有表）
详细不展开了。

## agenda
#### C-c a——
t 全局todo列表 T 自己选关键字
m 匹配tag和属性的标题
a 日历式列表（本周
L 当前文件时间轴视图
s 关键字/正则选中的条目列表
#### agenda下的命令
n 下一行 p 上一行
spc 在另一个窗口显示条目所在的位置
tab 类似 不选中（高亮）条目
ret 直接跳转到位置

o 删除其他窗口，比C-x o C-x 1块
. 跳到今天
j 跳到某一天
A = C-c a
r/g 更新agenda
t 更改todo状态

T show tags
： set tags
, 权重那边的

C-k 直接删除条目
C-c C-s 直接规划条目 C-d同理

I（大写i） 直接条目计时 
O stop /X cancel
J 跳到正在计时的窗口
####v view模式直接选择需要的视图(或者不记得某个键位了) 记得todo模式不支持视图
m 月 d/y/w同理
d/w 切换日周视图 b/f前一个后一个调整用
a 档案模式
R clockreport mode 意义不明
l log-mode（可能是最有用的一个）
c 据说是展示show overlapping（重叠）的clock 可是我这边怎么看都跟log-mode一样
总之见机行事（自己认下英文啦，明明提示都在那，和文档大概不是一个版本的，自己看吧）


行了，这样大概就不怎么需要再查文档了……遇到问题直接查英文文档问题应该也不大了。
大概看了下有点多，慢慢来吧（当初emacs不是也那么多嘛……用得多了就是肌肉记忆了）
