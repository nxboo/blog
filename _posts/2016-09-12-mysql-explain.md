---
layout: post
description: "Mysql性能分析"
---

#Mysql性能分析

执行mysql语句，只要在前面加上 explain 就可以分析查询语句的效率。

例如： 
> explain select focus_uid from user_focus where uid = xxx;

explain显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。

具体解释如下：


id 本次 select 的标识符。在查询中每个 select都有一个顺序的数值。  
select_type select 的类型，可能会有以下几种：
* simple: 简单的 select （没有使用 union或子查询）
* primary: 最外层的 select。
* union: 第二层，在select 之后使用了 union。
* dependent union: union 语句中的第二个select，依赖于外部子查询
* subquery: 子查询中的第一个 select
* dependent subquery: 子查询中的第一个 subquery依赖于外部的子查询
* derived: 派生表 select（from子句中的子查询）

table：显示这一行的数据是关于哪张表的  
type：这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、eq_ref、ref、range、index 和ALL  
system:表只有一行记录（等于系统表）。这是 const表连接类型的一个特例。  
const:表中最多只有一行匹配的记录，它在查询一开始的时候就会被读取出来。由于只有一行记录，在余下的优化程序里该行记录的字段值可以被当作是一个 恒定值。const表查询起来非常快，因为只要读取一次！const 用于在和 primary key 或unique 索引中有固定值比较的情形。下面的几个查询中，tbl_name 就是 c表了。  
eq_ref:从该表中会有一行记录被读取出来以和从前一个表中读取出来的记录做联合。与const类型不同的是，这是最好的连接类型。它用在索引所有部 分都用于做连接并且这个索引是一个primary key 或 unique 类型。eq_ref可以用于在进行"="做比较时检索字段。比较的值可以是固定值或者是表达式，表达示中可以使用表里的字段，它们在读表之前已经准备好 了。  
ref: 该表中所有符合检索值的记录都会被取出来和从上一个表中取出来的记录作联合。ref用于连接程序使用键的最左前缀或者是该键不是 primary key 或 unique索引（换句话说，就是连接程序无法根据键值只取得一条记录）的情况。当根据键值只查询到少数几条匹配的记录时，这就是一个不错的连接类型。 ref还可以用于检索字段使用 =操作符来比较的时候。  
ref_or_null: 这种连接类型类似 ref，不同的是mysql会在检索的时候额外的搜索包含null 值的记录。  
range: 只有在给定范围的记录才会被取出来，利用索引来取得一条记录。key字段表示使用了哪个索引。key_len字段包括了使用的键的最长部分。这种类型时 ref 字段值是 null。range用于将某个字段和一个定植用以下任何操作符比较时 =, <>, >,>=, <, <=, is null, <=>, between, 或 in。  
index: 连接类型跟 all 一样，不同的是它只扫描索引树。它通常会比 all快点，因为索引文件通常比数据文件小。mysql在查询的字段知识单独的索引的一部分的情况下使用这种连接类型。  
all: 将对该表做全部扫描以和从前一个表中取得的记录作联合。这时候如果第一个表没有被标识为const的话就不大好了，在其他情况下通常是非常糟糕的。正常地，可以通过增加索引使得能从表中更快的取得记录以避免all。  


得保证查询至少达到range级别，最好能达到ref，否则就可能会出现性能问题。  


possible_keys：显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句  
key：    实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用force  INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引  
key_len：使用的索引的长度。在不损失精确性的情况下，长度越短越好  
ref：    显示索引的哪一列被使用了，如果可能的话，是一个常数  
rows：   MYSQL认为必须检查的用来返回请求数据的行数  
Extra：  关于MYSQL如何解析查询的额外信息。将在表4.3中讨论，但这里可以看到的坏的例子是Using temporary和Using filesort，意思MYSQL根本不能使用索引，结果是检索会很慢  

extra列返回的描述的意义  
Distinct:一旦MYSQL找到了与行相联合匹配的行，就不再搜索了  
Not exists: MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了  
Range checked for each Record（index map:#）:没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一  

Using filesort: mysql需要额外的做一遍从而以排好的顺序取得记录。排序程序根据连接的类型遍历所有的记录，并且将所有符合 where条件的记录的要排序的键和指向记录的指针存储起来。这些键已经排完序了，对应的记录也会按照排好的顺序取出来。  

Using index: 列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候  
using temporary: mysql需要创建临时表存储结果以完成查询。这种情况通常发生在查询时包含了groupby 和 order by 子句，它以不同的方式列出了各个字段。  
using where

where子句将用来限制哪些记录匹配了下一个表或者发送给客户端。除非你特别地想要取得或者检查表种的所有记录，否则的话当查询的extra 字段值不是 using where 并且表连接类型是 all 或 index时可能表示有问题。  

如果你想要让查询尽可能的快，那么就应该注意 extra 字段的值为usingfilesort 和 using temporary 的情况。  
