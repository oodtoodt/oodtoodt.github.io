---
title: MySQL必知必会-notes-1
date: 2020-04-13 17:01:36
tags:
---

ongoing。

<!--more-->

## 主键
一列，其值能唯一区分表中每行
+ 任意两行不具有相同的主键值
+ 每行必须具有一个主键值
主键通常就是一列，也可能是几列
    普遍的习惯：
    不更新主键列中的值
    不重用它
    不再主键列使用可能会更改的值

## 什么SQL
结构化查询语言（Structured Query Language）
注意SQL不一定完全可移植

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