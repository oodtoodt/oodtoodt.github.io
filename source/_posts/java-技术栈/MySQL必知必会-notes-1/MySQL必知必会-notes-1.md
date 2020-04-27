---
title: MySQL必知必会-notes-1
date: 2020-04-13 17:01:36
tags:
- DB
- mysql
- company

categories:
- DB
- mysql
- book_notes

---

ongoing。

<!--more-->
<!-- TOC -->

- [主键](#%E4%B8%BB%E9%94%AE)
- [什么是SQL](#%E4%BB%80%E4%B9%88%E6%98%AFsql)
- [注意](#%E6%B3%A8%E6%84%8F)
    - [注释](#%E6%B3%A8%E9%87%8A)
    - [删除](#%E5%88%A0%E9%99%A4)
    - [大小写](#%E5%A4%A7%E5%B0%8F%E5%86%99)
- [检索](#%E6%A3%80%E7%B4%A2)
    - [检索列](#%E6%A3%80%E7%B4%A2%E5%88%97)
    - [检索不同的行](#%E6%A3%80%E7%B4%A2%E4%B8%8D%E5%90%8C%E7%9A%84%E8%A1%8C)
    - [注意](#%E6%B3%A8%E6%84%8F)
- [排序](#%E6%8E%92%E5%BA%8F)
- [过滤条件](#%E8%BF%87%E6%BB%A4%E6%9D%A1%E4%BB%B6)
    - [组合where](#%E7%BB%84%E5%90%88where)
        - [IN](#in)
        - [Not](#not)
- [通配符过滤](#%E9%80%9A%E9%85%8D%E7%AC%A6%E8%BF%87%E6%BB%A4)
    - [like](#like)
        - [%](#%25)
        - [](#)
- [正则搜索](#%E6%AD%A3%E5%88%99%E6%90%9C%E7%B4%A2)
    - [基本字符匹配](#%E5%9F%BA%E6%9C%AC%E5%AD%97%E7%AC%A6%E5%8C%B9%E9%85%8D)
    - [OR匹配](#or%E5%8C%B9%E9%85%8D)
    - [匹配几个之一](#%E5%8C%B9%E9%85%8D%E5%87%A0%E4%B8%AA%E4%B9%8B%E4%B8%80)
    - [匹配范围](#%E5%8C%B9%E9%85%8D%E8%8C%83%E5%9B%B4)
    - [特殊符](#%E7%89%B9%E6%AE%8A%E7%AC%A6)
    - [匹配字符类](#%E5%8C%B9%E9%85%8D%E5%AD%97%E7%AC%A6%E7%B1%BB)
    - [匹配多个实例](#%E5%8C%B9%E9%85%8D%E5%A4%9A%E4%B8%AA%E5%AE%9E%E4%BE%8B)
    - [定位符](#%E5%AE%9A%E4%BD%8D%E7%AC%A6)
- [创建计算字段](#%E5%88%9B%E5%BB%BA%E8%AE%A1%E7%AE%97%E5%AD%97%E6%AE%B5)
    - [concat(concatenate)](#concatconcatenate)
    - [as](#as)
    - [执行算术计算](#%E6%89%A7%E8%A1%8C%E7%AE%97%E6%9C%AF%E8%AE%A1%E7%AE%97)
- [使用数据处理函数](#%E4%BD%BF%E7%94%A8%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E5%87%BD%E6%95%B0)
- [汇总数据](#%E6%B1%87%E6%80%BB%E6%95%B0%E6%8D%AE)
    - [聚集函数](#%E8%81%9A%E9%9B%86%E5%87%BD%E6%95%B0)
    - [聚集不同值](#%E8%81%9A%E9%9B%86%E4%B8%8D%E5%90%8C%E5%80%BC)
- [分组数据](#%E5%88%86%E7%BB%84%E6%95%B0%E6%8D%AE)
- [子查询](#%E5%AD%90%E6%9F%A5%E8%AF%A2)
    - [计算的子查询](#%E8%AE%A1%E7%AE%97%E7%9A%84%E5%AD%90%E6%9F%A5%E8%AF%A2)
- [联结表](#%E8%81%94%E7%BB%93%E8%A1%A8)
- [高级联结](#%E9%AB%98%E7%BA%A7%E8%81%94%E7%BB%93)
    - [使用表别名](#%E4%BD%BF%E7%94%A8%E8%A1%A8%E5%88%AB%E5%90%8D)
    - [使用不同联结](#%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%90%8C%E8%81%94%E7%BB%93)
    - [使用联结和联结条件](#%E4%BD%BF%E7%94%A8%E8%81%94%E7%BB%93%E5%92%8C%E8%81%94%E7%BB%93%E6%9D%A1%E4%BB%B6)
- [组合查询](#%E7%BB%84%E5%90%88%E6%9F%A5%E8%AF%A2)
    - [组合查询](#%E7%BB%84%E5%90%88%E6%9F%A5%E8%AF%A2)
    - [创建组合](#%E5%88%9B%E5%BB%BA%E7%BB%84%E5%90%88)
- [全文本搜索](#%E5%85%A8%E6%96%87%E6%9C%AC%E6%90%9C%E7%B4%A2)
    - [为什么需要](#%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81)
    - [使用全文本搜索](#%E4%BD%BF%E7%94%A8%E5%85%A8%E6%96%87%E6%9C%AC%E6%90%9C%E7%B4%A2)
        - [启动](#%E5%90%AF%E5%8A%A8)
        - [使用查询扩展](#%E4%BD%BF%E7%94%A8%E6%9F%A5%E8%AF%A2%E6%89%A9%E5%B1%95)
        - [布尔文本搜索](#%E5%B8%83%E5%B0%94%E6%96%87%E6%9C%AC%E6%90%9C%E7%B4%A2)
- [数据插入](#%E6%95%B0%E6%8D%AE%E6%8F%92%E5%85%A5)
- [更新数据](#%E6%9B%B4%E6%96%B0%E6%95%B0%E6%8D%AE)
- [删除数据](#%E5%88%A0%E9%99%A4%E6%95%B0%E6%8D%AE)
- [更新删除的原则](#%E6%9B%B4%E6%96%B0%E5%88%A0%E9%99%A4%E7%9A%84%E5%8E%9F%E5%88%99)
- [创建表](#%E5%88%9B%E5%BB%BA%E8%A1%A8)
    - [语句格式化](#%E8%AF%AD%E5%8F%A5%E6%A0%BC%E5%BC%8F%E5%8C%96)
    - [使用null值](#%E4%BD%BF%E7%94%A8null%E5%80%BC)
    - [主键](#%E4%B8%BB%E9%94%AE)
    - [使用autoincrement](#%E4%BD%BF%E7%94%A8autoincrement)
    - [指定默认值default](#%E6%8C%87%E5%AE%9A%E9%BB%98%E8%AE%A4%E5%80%BCdefault)
    - [引擎](#%E5%BC%95%E6%93%8E)
    - [更新表](#%E6%9B%B4%E6%96%B0%E8%A1%A8)
    - [重命名表](#%E9%87%8D%E5%91%BD%E5%90%8D%E8%A1%A8)
- [使用视图](#%E4%BD%BF%E7%94%A8%E8%A7%86%E5%9B%BE)
    - [视图的作用](#%E8%A7%86%E5%9B%BE%E7%9A%84%E4%BD%9C%E7%94%A8)
    - [视图的规则和限制](#%E8%A7%86%E5%9B%BE%E7%9A%84%E8%A7%84%E5%88%99%E5%92%8C%E9%99%90%E5%88%B6)
    - [使用视图](#%E4%BD%BF%E7%94%A8%E8%A7%86%E5%9B%BE)
    - [利用视图简化复杂的联结](#%E5%88%A9%E7%94%A8%E8%A7%86%E5%9B%BE%E7%AE%80%E5%8C%96%E5%A4%8D%E6%9D%82%E7%9A%84%E8%81%94%E7%BB%93)
    - [用视图重新格式化检索出的数据](#%E7%94%A8%E8%A7%86%E5%9B%BE%E9%87%8D%E6%96%B0%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%A3%80%E7%B4%A2%E5%87%BA%E7%9A%84%E6%95%B0%E6%8D%AE)
    - [视图与计算字段](#%E8%A7%86%E5%9B%BE%E4%B8%8E%E8%AE%A1%E7%AE%97%E5%AD%97%E6%AE%B5)
    - [更新视图](#%E6%9B%B4%E6%96%B0%E8%A7%86%E5%9B%BE)
- [存储过程](#%E5%AD%98%E5%82%A8%E8%BF%87%E7%A8%8B)
    - [为什么](#%E4%B8%BA%E4%BB%80%E4%B9%88)
    - [使用存储过程](#%E4%BD%BF%E7%94%A8%E5%AD%98%E5%82%A8%E8%BF%87%E7%A8%8B)
    - [使用参数](#%E4%BD%BF%E7%94%A8%E5%8F%82%E6%95%B0)
    - [智能存储过程](#%E6%99%BA%E8%83%BD%E5%AD%98%E5%82%A8%E8%BF%87%E7%A8%8B)
- [游标](#%E6%B8%B8%E6%A0%87)
    - [创建、打开、关闭、使用](#%E5%88%9B%E5%BB%BA%E6%89%93%E5%BC%80%E5%85%B3%E9%97%AD%E4%BD%BF%E7%94%A8)
- [触发器](#%E8%A7%A6%E5%8F%91%E5%99%A8)
- [事务处理](#%E4%BA%8B%E5%8A%A1%E5%A4%84%E7%90%86)
    - [ROLLBACK](#rollback)
    - [COMMIT](#commit)
- [全球化和本地化](#%E5%85%A8%E7%90%83%E5%8C%96%E5%92%8C%E6%9C%AC%E5%9C%B0%E5%8C%96)
- [安全管理](#%E5%AE%89%E5%85%A8%E7%AE%A1%E7%90%86)

<!-- /TOC -->


## 主键
一列，其值能唯一区分表中每行
+ 任意两行不具有相同的主键值
+ 每行必须具有一个主键值
主键通常就是一列，也可能是几列
    普遍的习惯：
    不更新主键列中的值
    不重用它
    不再主键列使用可能会更改的值

## 什么是SQL
结构化查询语言（Structured Query Language）
注意SQL不一定完全可移植

## 注意

### 注释
用杠杠的时候给爷tmd加上空格！不加不认识！

### 删除
+ drop主要用于删除结构
例如删除数据库：drop database XX，删除表 drop table XX。字段也是结构的一种，也可以使用drop了？对的，但是我们改变了表结构要先alter方法。例如，我们要删除student表上的age字段的信息，可以这样写：alter table student drop age

+ delete主要用于删除数据
举个例子，要删除 student表上名字为‘张三’的所有信息：delete from student where name=‘张三’。这种情况下用delete，由此可见delete常用于删除数据。

### 大小写
注意在linux下会有大小写的区分
windows下全部不区分大小写

查询字符串对大小写不敏感

关键字和函数名不区分大小写
存储函数、存储过程、事件的名字不区分大小写，触发器区分
数据列和索引的名字不区分

不能完全保证，仅供参考

## 检索
help show...show可以帮助你做一些事。

### 检索列
select ... from ...
>SQL语句一般返回原始的、无格式的数据。数据的格式化是一个表示问题，而不是一个检索问题
其中，前面的...可以填单个列，可以填多个列(用逗号隔开)，可以填星号(`*`)通配符匹配所有列

### 检索不同的行
```java
SELECT DISTINCT vend_id FROM products
```
注意不能部分使用DISTINCT

### 注意
行从0开始
建议使用LIMIT 4 OFFSET 3，即LIMIT 3,4，但是更显然：从行3开始取4行

## 排序
order by
怎么样有没有想起一些？
多个按顺序
DESC 降序

## 过滤条件
where可以检索需要的行
where prod_price BETWEEN 5 AND 10;
where .. is null(注意排除not null的行并不能得到null的行)

### 组合where
AND
OR

#### IN 
Where vend_id IN (1002,1003)

#### Not
where vend_id not in (1002,1003)

## 通配符过滤

### like

#### %
```sql
select prod_id, prod_name
from products
where prod_name like 'jet%';
```
%表出现任意次数，这里指jet开头的所有串

#### _
匹配单个字符

## 正则搜索

### 基本字符匹配
REGEXP 后跟正则
```sql
where prod_name REGEXP '.000'
```
.表示任意字符
>like是严格匹配的，REGEXP是部分匹配的

**匹配不区分大小写**,可以使用binary来区分

### OR匹配
`|`为正则的or符

### 匹配几个之一
`[123]` == `[1|2|3]`

### 匹配范围
`[1-5]`

### 特殊符
`\\-`表示查找`-`，即转义

### 匹配字符类
举个例子：`[:alnum:]`匹配任意字母和数字。
具体请自查。

### 匹配多个实例
```sql
* 0或多个
+ 1或多个
? 0或1个
-- 下面是给前面做工作的
{n} 指定n个匹配
{n,} 下限n个
{n,m} n-m个，不超过255
```

### 定位符
```sql
^ 文本开始
$ 文本结束
[[:<:]] 词的开始
[[:>:]] 词的结束
--比如
where prod_name regexp '^[0-9\\.]'
```
即开头是0-9或者是.的才会被匹配

## 创建计算字段

### concat(concatenate)
```sql
 select Concat(RTrim(vend_name),' (', RTrim(vend_country), ')')
    -> from vendors
    -> order by vend_name
    -> ;
```
可以把多个串连接成一个较长的串
RTrim()函数可以去掉值右边的所有空格

### as
命名一个别名

### 执行算术计算
```sql
+---------+----------+------------+
| prod_id | quantity | item_price |
+---------+----------+------------+
| ANV01   |       10 |       5.99 |
| ANV02   |        3 |       9.99 |
| TNT2    |        5 |      10.00 |
| FB      |        1 |      10.00 |
+---------+----------+------------+
mysql> select prod_id,
    -> quantity,
    -> item_price,
    -> quantity * item_price AS expanded_price
    -> from orderitems
    -> where order_num = 20005;
+---------+----------+------------+----------------+
| prod_id | quantity | item_price | expanded_price |
+---------+----------+------------+----------------+
| ANV01   |       10 |       5.99 |          59.90 |
| ANV02   |        3 |       9.99 |          29.97 |
| TNT2    |        5 |      10.00 |          50.00 |
| FB      |        1 |      10.00 |          10.00 |
+---------+----------+------------+----------------+
```

## 使用数据处理函数
Upper()
RTrim()...等等请查表。这里只讲几个
soundex是一个将任何文本串转换为描述其语音表示的字母数字模式的算法
日期的、数据的、文本的都有

## 汇总数据

### 聚集函数
AVG(),COUNT(),MAX(),MIN(),SUM()
如果`count(*)`，则不忽略空行
MAX(),MIN()要求指定列名

### 聚集不同值
+ 对所有的行执行计算，指定ALL参数或不给参数（all是默认行为）
+ 只包含不同的值，指定DISTINCT参数
DISTINCT不能用于`count(*)`

可以有多个聚集函数

## 分组数据
group by，可以对行计数，计算和平均数，获得最大最小值而不用检索所有数据——
【不用检索所有数据，这里是比较关键的点，就是说你需要的只是在几列里面检索几行，不需要其他的那些东西
Having可以过滤分组，用法和where一模一样
Group by和order by经常完成相同的工作，但他们是非常不同的,记得在group by后面跟order by

这时候可以回顾一下select的顺序了：
select  必要
from  表中必要
where 非
group by 非
having 非
order by 非
limit(要检索的行数) 非

## 子查询
即嵌套
```sql
 select cust_id 
 from orders 
 where order_num in (select order_num 
                     from orderitems 
                     where prod_id = 'TNT2');
+---------+
| cust_id |
+---------+
|   10001 |
|   10004 |
+---------+
2 rows in set (0.00 sec)
```
由于性能的限制，不能嵌套太多的子查询

### 计算的子查询
指的是子查询中有
```sql
in (select ...
from ...
where orders.cust_id = coustomers.cust_id)
```

## 联结表
联结（join）
创建联结只需要规定表和他们之间如何关联即可
【你可以把两个表直接想象成笛卡尔积，从这里面使用过滤
可以用where作为过滤，比如where a = b
内部联结——INNER JOIN

但是需要注意的是，一般而言子查询效率不如联结的效率，因为子查询的默认是笛卡尔积，而联结只是表遍历+表遍历

## 高级联结

### 使用表别名
```sql
select cust_name, cust_contact
from customers as c,orders as o, orderitems as oi
where c.cust_id = o.cust_id
and oi.order_num = o.order_num
and prod_id = 'TNT2';
```

### 使用不同联结
+ 自联结
```sql
self as p1,self as p2
where p1.X = p2.X
```
+ 自然联结
自然联结排除多次出现，使每个列只返回一次。
然而这序言自己做，你只选择那些唯一的列(对某一张表通配符)，然后对其他表的列使用明确的子集。
```sql
select c.*,o.order_num
from customers as c,orders as o
where c.cust_id = o.cust_id;
```
+ 外部联结
需要包含没有关联行的那些行
比如
>你需要对客户下了多少订单进行计数，包括那些至今尚未下单的客户
>列出所有产品和订购数量，包括没人订购的
```sql
select customers.cust_id,orders.order_num
from customers inner join orders
on customers.cust_id = orders.cust_id;
```

### 使用联结和联结条件
+ 一般使用内部联结，外部联结也是有效的
+ 保证正确联结条件
+ 总是提供联结条件（不然会退化为笛卡尔积）
+ 一个联结中可以包含多个表，甚至对每个联结可以采用不同的联结类型

## 组合查询

### 组合查询
执行多个select语句，并将结果作为单个结果返回
+ 在单个查询中从不同的表返回类似结构的数据
+ 对单个表执行多个查询，按单个查询返回数据

### 创建组合
可用union来组合数条查询
只需在不同的select之间使用union即可
union中必须包含相同的列、表达式或聚集函数
```sql
select vend_id, prod_id, prod_price
from products
where prod_price <= 5
union
select vend_id, prod_id, prod_price
from products
where vend_id in (1001,1002);
```
可以使用union all来匹配所有行，不取消重复的行

## 全文本搜索
并发所有的引擎都支持全文本搜索，比如MyISAM支持而InnoDB不支持

### 为什么需要
有了like和正则，为什么还需要全文本搜索？
+ 性能
+ 明确控制，两者很难明确地控制匹配什么和不匹配什么——比如第一个词必须匹配，第二个词必须不匹配，第三个词必须在第一个词确实匹配的情况下才匹配或不匹配
+ 智能化 两者都无法提供智能化选择结果的方法——比如不能区分包含单个匹配的行和包含多个匹配的行。

全文本搜索会创建指定列中各词的一个索引，搜索针对这些词进行。

### 使用全文本搜索

#### 启动
一般在创建表时启用，create table接受fulltext子句
```sql
CREATE TABLE productnotes
(
    note_id int not NULL auto_increment,
    prod_id char(10) not null,
    ...
    primary key(note_id),
    fulltext(note_text)
)engine = myisam;

select note_next
from productnotes
where match(note_text) against('rabbit');
```
比起like的区别是全文本搜索可以按等级排序，比如这里会把rabbit做第三个词的行排在做第20个词的前面。
等级由行词的数目、唯一词的数目、整个索引中词的总数以及包含该词的行的数目计算出来

#### 使用查询扩展
with QUERY expansion
会返回一些和你的搜索有关的所有其他行
比如你的搜索match against('anvils')只有一行结果，但是扩展之后就会有很多：第一行包含了anvils;包含了第一行的几个词的行也会被检索出来，依照位置等信息等级排序

#### 布尔文本搜索
+ 要匹配的词
+ 要排斥的词
+ 排列提示(指定某些词等级更高)
+ 表达式分组
+ 另外一些内容
即便没有fulltext也可以使用

## 数据插入
插入一行
插入一行的一部分
插入多行
插入某些查询结果
> 可针对每个表，利用mysql的安全机制禁止insert

>insert一般不会有输出

一般可以直接插入：
```sql
insert into Table
values(
    ...
);
```
但很不安全,安全的话应当先对每一个值指定一个列名，即使表的顺序结构改变仍可以正常工作。

处理请求和按什么次序处理时mysql的任务。insert可能很耗时（尤其是需要更新许多索引时）
如果数据检索是最重要的，那么你可以降低insert的优先级，`insert low_priority into`

可以使用多条insert，也可以一次提交它们(列名相同)

可以把value(..)替换成`select ... from ... where ...`

## 更新数据
update，可以用于更新特定行，或是所有航。
>一定更要注意细心，稍不注意就会更新所有行，不要省略where

>可以限制和控制update的使用

```sql
update customers
set cust_email = 'elmer@fudd.com'
where cust_id = 10005
```

update中可以使用子查询
更新多行如果出错，整个更新将被取消

## 删除数据
delete注意事项同更新
甚至使用起来更容易
无需set，直接delete from ... where ...，不需指定列或通配符——删除整行而不删除列，为删除列，请使用update

## 更新删除的原则
+ 除非确实打算更新/删除**每**一行，不然决不要使用不带where的update或delete
+ 保证每个表都有主键，尽可能像where子句那样使用它(可以指定各主键，多个值)
+ 在对update或delete语句使用where子句前，应该先用select进行测试，保证他过滤的是正确的记录
+ 使用强制实施引用完整性的数据库，不允许删除具有和其他表相关联的数据的行

>引用完整性通过在表的定义中指定主键和外键来实现的，后面会讲

**MySQL没有撤销按钮！**

## 创建表

### 语句格式化
+ mysql语句中忽略空格
+ 语句可以在一个长行中输入，也可以分成多行
+ 使用恰当的缩进（sql中没有缩进的规定）

### 使用null值
即not null/ null
>不要把null值和空串混淆

### 主键
主键值必须唯一。如果使用多个列，那么这些列的组合值必须唯一

### 使用auto_increment
告诉mysql，本列每当增加一行时自动增量
每个表只允许一个auto_increment列，必须被索引
确定值可以使用 `select last_insert_id()`函数获得

### 指定默认值default
不允许使用函数作为默认值，只支持常量

### 引擎
可以混用，但外键不能跨引擎
具体情况需要具体分析

### 更新表
alter table可以更新表的定义
**但是理想状态下当表中存储数据以后，该表就不应该再被更新**

```sql
alter table vendors
add vend_phone char(20);

alter table vendors
drop column vend_phone;
```

alter table一种常见用途是定义外键：
```sql
atler tabl eorders
add constraint fk_orders_customers foreign key(cust)id
references customers(cust_id)
```

>使用alter table要极为小心，应当事先备份。

复杂的表结构更改一般需要手动删除过程，涉及以下步骤：
+ 新的列布局创建新表
+ 使用insert select语句
+ 检验包含所需数据的旧表
+ 重命名旧表
+ 用旧表原来的名字重命名新表
+ 根据需要，重建触发器、存储过程、索引、外键

### 重命名表 
rename table ... to

## 使用视图
视图是虚拟的表，只包含使用时动态检索数据的查询

### 视图的作用
+ 重用sql语句
+ 简化复杂的sql操作，在编写查询后，可以方便地重用它而不必知道它的基本查询细节
+ 使用表的组成部分而不是整个表
+ 保护数据——可以授予特定部分的访问权限
+ 更改数据格式和表示——视图可以返回与底层表的表示和格式不同的数据

>如果你用多个联结和购率创建了复杂的视图或嵌套了视图，可能会发现性能下降的很厉害。

### 视图的规则和限制
+ 唯一命名
+ 可以创建的视图的数目没有限制
+ 创建视图必须有足够的访问权限
+ 视图可以嵌套，可以用从其他视图中检索数据的查询来构造一个视图
+ order by可以用在视图中。select的order by可以覆盖它。
+ 视图不能索引，也不能有关联的触发器或默认值
+ 视图可以和表一起用，比如联结表和视图

### 使用视图
+ 用create view来创建
+ 使用show create view viewname来查看创建视图的语句
+ 用drop view viewname删除视图
+ 更新视图时，可以先drop再create，也可以直接`create or replace view`

### 利用视图简化复杂的联结
```sql
create view productcustomers as
select cust_name, cust_contact, prod_id
from customers, orders, orderitems
where customers.cust_id = orders.cust_id
and orderitems.order_num = orders.order_num;
```
创建一个名为productcustomers的视图，联结三个表，以返回已订购了任意产品的所有客户的列表
为检索订购了产品TNT2的客户，可
```sql
select cust_name, cust_contact
from productcustomers
where prod_id = 'TNT2';
```
由此可以大大简化sql语句的使用
*创建可重用的视图* 创建不受特定数据限制的视图是一种好方法，例如上面的视图返回生成所有商品的客户而不仅仅是TNT2的客户。

### 用视图重新格式化检索出的数据
简单的例子就是如果经常需要某个结果，不必在每次需要时执行联结，创建一个视图，每次需要时使用它即可
同样适用于某些需要每次都要where的语句（比如not null)

注如果视图里用了一条where，而视图原来自带一条where，它们将自动组合

### 视图与计算字段
视图对于简化计算字段的使用特别有用

比如你需要`select 一大堆 xx*yy as ... from ... where...`来计算（比如每种物品的总价格）
就可以转换为一个视图`create view ... as select ... xx*yy as ... from ..;`
之后需要检索订单20005的各项物品总价格只需要`select * from VIEWNAME where order_num = 20005;`

### 更新视图
视图的数据能否更新？答案是情况而定
通常，它是可更新的。如果更新一个视图，你将更新到基表上
如果MySQL不能正确地确定被更新的基数据，则不允许更新(包括插入和删除)。这意味着如果视图有下列操作则无法更新：
+ 分组(group by 和 having)
+ 联结
+ 子查询
+ 并
+ 聚集函数（min(),count(),sum()）
+ distinct
+ 导出(计算)列

>这个限制很可能会在未来(现在)的版本中渐渐放开。

一般应当将视图用于检索（select语句）而不用于更新

## 存储过程

### 为什么
+ 通过把处理封装在容易使用的单元中，简化复杂的操作
+ 保证了数据的完整性，使用同一存储过程则使用的代码都是相同的
+ 简化对变动的管理——延伸出安全性
+ 提高性能。存储过程可以使用只能用在单个请求中的mysql元素和特性来编写更灵活更强的代码。
总的说，简单、安全、高性能。
缺陷也是有的：
+ 比基本sql复杂，需要更高的技能和经验
+ 你可能没有创建存储过程的安全访问权限

### 使用存储过程
```sql
call productpricing(@pricelow,
                    @pricehigh,
                    @priceaverage);
```
调用。怎么样，有趣吧。

创建
```sql
create procedure XX()
begin
    select .. as ..
    from ..;
end;
```
删除就`drop procedure XX;`

### 使用参数
```sql
create procedure productpricing(
    out pl DECIMAL(8,2),
    out ph DECIMAL(8,2),
    out pa DECIMAL(8,2)
)
begin
    select min(prod_price)
    into pl
    from products;
    select max(prod_price)
    into ph
    from products;
    select avg(prod_price)
    into pa
    from products;
end;
```
接受3个参数：pl,ph,pa，out指出是传出给调用者，in传递给存储过程

不能通过一个参数返回多个行和列

调用完了call之后就可以
```sql
select @pricehigh, @pricelow, @priceaverage;
```

### 智能存储过程
```sql
-- 创建存储过程
delimiter $$
-- //使用$$作为存储过程的分隔符
create procedure ordertotal(
    in onumber int,
    in taxable boolean,
    out ototal decimal(8,2)
)comment 'Obtain order total,optionally adding tax'
begin 
    declare total decimal(8,2);
    -- declare variable for total
    declare taxrate int default 6;  -- 声明存储过程中的变量

    select sum(item_price*quantity) 
    from orderitems
    where order_num=onumber 
    into total;

    if taxable then
        select total+(total/100*taxrate) into total;
    end if;
    -- and finally save to out variable
    select total into ototal;
end;$$
delimiter ;
-- 调用存储过程
call ordertotal(20005,1,@total);
-- 显示结果
select @total;
-- 删除存储过程
drop procedure ordertotal;
```
这里我抄的网上的代码，他这个分隔符看的我心惊胆战的(被我改了)

comment会在show procedure status的结果中显示
```sql
show create procedure ordertotal;
show procedure status like 'ordertotal';
```

## 游标
游标主要用于交互式应用，其中用户需要滚动屏幕上的数据，并对数据进行浏览或更改。
mysql游标只能用于存储过程(和函数)。
+ 使用前必须定义它
+ 声明后必须*打开*以供使用
+ 对填有数据的游标，根需取行
+ 结束使用必须关闭

### 创建、打开、关闭、使用
```sql
delimiter #
CREATE procedure processorders()
begin

    -- Declare local variables
    DECLARE done BOOLEAN default 0;
    DECLARE o INT;
    DECLARE t DECIMAL(8,2);

    -- Declare the cursor
    DECLARE ordernumbers CURSOR
    FOR
    SELECT order_num FROM orders;

    -- Declare continue handler
    DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;

    -- Create a table to store the results
    CREATE TABLE IF NOT EXISTS ordertotals
        (order_num INT, total DECIMAL(8,2));

    -- Open the cursor
    OPEN ordernumbers;

    -- Loop through all rows
    REPEAT
        
        -- Get order number
        FETCH ordernumbers INTO o;
        
        -- Get the total for this order
        -- 就是上一章的函数
        CALL ordertotal(o,1,t);

        -- Insert order and total into ordertotals
        Insert into ordertotals(order_num, total)
        VALUES(o,t);

    -- End of loop
    UNTIL done END REPEAT;

    -- Close the cursor
    CLOSE ordernumbers;
    
END;#

delimiter ;

```
此存储过程不返回数据，但它能够创建和填充另一个表。用CALL执行了另一个存储过程来计算并用Insert存储下来。
坑：拼写，`--`后面跟空格，错误会显示出来，就在里面之中。

## 触发器
如果你想要某条语句在事件发生时自动执行，就需要触发器
+ 唯一的触发器名
+ 触发器关联的表
+ 触发器相应的活动（delete、insert、update）
+ 触发器何时执行

```sql
CREATE TRIGGER neworder AFTER INSERT ON orders
FOR EACH ROW SELECT NEW.order_num into @ee;
```
**不推荐使用**

## 事务处理
MySQL 事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！
1. 在MySQL 中只有使用了Innodb数据库引擎的数据库或表才支持事务
2. 事务处理可以用来维护数据库的完整性，保证成批的SQL 语句要么全部执行，要么全部不执行
3. 事务用来管理 insert,update,delete 语句

### ROLLBACK
```sql
SELECT * FROM customers;# 先检索一下当前表中的数据

# 以下三行选中一块执行
START TRANSACTION;#开始执行一个事务
DELETE FROM customers;#删除表中数据，需要注意的是，如果要删除表中的数据，这个表和其他表不要有关联关系，否则报错，这里删除customers表就报错，换成一个单独的表，可以正常执行
SELECT * FROM customers;#检查一下表中的数据是否被删除，正常是被清空的

ROLLBACK;回退到START TRANSACTION之前的情形
SELECT * FROM customers;#再次检查发现表中又有数据了
```

### COMMIT

```sql
# 从系统完全删除订单20010，因为涉及到删除两个数据表中的内容，所以使用事务块处理来保证订单不被部分删除，
START TRANSACTION;
DELETE FROM orderitems WHERE order_num=20010;
DELETE FROM orders WHERE order_num=20010;
COMMIT;#最后commit语句只在不出错的情况下提交更改，事务块中的任何一个语句错误，都不会做出更改
```
使用ROLLBACK和COMMIT就可以写入或撤销整个事务处理，但是只能对简单的事务处理才能这么做，更复杂的事务处理可能需要部分提交或回退。

savepoint 是在数据库事务处理中实现“子事务”（subtransaction），也称为嵌套事务的方法。事务可以回滚到 savepoint 而不影响 savepoint 创建前的变化, 不需要放弃整个事务。
```sql
SAVEPOINT point1;    // 声明一个savepoint

ROLLBACK TO point1;  // 回滚到point1
```

## 全球化和本地化
+ 字符集为字母和符号的集合
```sql
show character set;-- 字符集
show collation;-- 校对
show variables like 'collation%';
-- 分割线
create table mytable
(...)defalut character set hebrew
collate hebrew_general_ci;

```
编码为某个字符集成员的内部表示
校对为规定字符如何比较的指令


## 安全管理
```sql
creater user ben identified by 'p@$$wOrd';
```
设置权限使用GRANT语句，其反操作为REVOKE。
可以做关于未来的授权
