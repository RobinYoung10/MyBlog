---
title: MySQL索引
date: 2019-04-27 19:17:37
tags: MySQL、索引
---

索引用于快速找出在某个列中有一特定值的行。不使用索引，MySQL必须从第1条记录开始读完整个表，直到找出相关的行。表越大，查询数据花费的时间越多。如果查询的列中有一个索引，MySQL则能快速到达某个位置，而不必查看所有数据。

### 一、索引简介

索引是一个单独的、存储在磁盘上的数据库结构，他们包含着对数据表里所有记录的引用指针。所有MySQL列类型都可以被索引。

##### 索引的优点和缺点

优点：

1. 可以大大加快数据的查询速度。
2. 在使用分组和排序子句进行查询数据时，可以显著减少查询中分组和排序的时间。

缺点：

1. 创建索引和维护索引需要耗费时间，而且随着数据量的增加所耗费时间增加。
2. 索引需要占用一定的磁盘空间。
3. 对表进行增加、删除和修改时，索引也需要动态的维护，降低了数据维护速度。

##### 索引的分类

1. 普通索引：基本的索引类型，允许重复值和空值。
2. 唯一索引：索引列的值必须唯一，但允许为空值。
3. 单列索引：一个索引只包含单个列，一个表可以有多个单列索引。
4. 组合索引：在表的多个字段组合上创建索引，只有在查询条件中使用了这些字段的左边字段时，才会被使用。
5. 全文索引：类型为FULLTEXT，在定义索引的列上支持值的全文查找，允许重复值和空值。可以在char、varchar或者text类型的列创建。
6. 空间索引：是对空间数据类型的字段建立的索引。

##### 索引的设计原则

1. 索引并非越多越好，一个表中如有大量的索引，不仅占用磁盘空间，而且还会影响增、删和改等语句的性能。
2. 避免对经常更新的表进行过多的索引，并且索引中的列尽可能少。
3. 数据量小的表最好不要使用索引，由于数据少，索引可能不会产生优化效果。
4. 在条件表达式中经常用到的不同值较多的列上建立索引，在不同值很少的列上不要建立索引。比如“性别”字段只有“男”和“女”两个值，无需建立索引。
5. 在频繁进行排序或分组的列上建立索引。

### 二、创建索引

##### 在创建表时创建索引

###### 1.创建普通索引

```sql
create table t1 (
id int not null,
name char(30) not null,
index(id));
```

使用explain语句查看索引是否在使用：

```sql
mysql> explain select * from t1 where id=111 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: ref
possible_keys: id
          key: id
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

possible_keys给出可选用的索引，key时MySQL实际选用的索引。

###### 2.创建唯一索引

```sql
create table t2 (
id int not null,
name char(30) not null,
unique index UniqIdx(id)
);
```

###### 3.创建单列索引

前面两个例子其实创建的索引都为单列索引。下面创建索引长度为20的单列索引。

```sql
create table t3 (
id int not null,
name char(30) not null,
index SingleIdx(name(20))
);
```

###### 4.创建组合索引

组合索引是在多个字段上创建索引。

```sql
create table t4 (
id int not null,
name char(30) not null,
age int not null,
info varchar(255),
index MultiIdex(id, name, age)
);
```

组合索引遵从“最左前缀”：利用索引中最左边的列集来匹配行。在这个例子里面如果查询条件组合为：（id，name，age）、（id，name）或者id，则会使用索引，否则不能。如下

```sql
mysql> explain select * from t4 where id = 1 and name = 'robin' \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t4
   partitions: NULL
         type: ref
possible_keys: MultiIdex
          key: MultiIdex
      key_len: 94
          ref: const,const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

```sql
mysql> explain select * from t4 where name = 'robin' \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t4
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

###### 5.创建全文索引

全文索引可用于全文搜索，适合大型数据集。

```sql
create table t5 (
id int not null,
name char(30) not null,
age int not null,
info varchar(255),
fulltext index FullTxtIdx(info)
)engine=MyISAM;
```

###### 5.创建空间索引

```sql
create table t6 (
g geometry not null, spatial index SpatIdx(g)
)engine = MyISAM;
```

##### 在已存在的表创建索引

在t表中id字段创建名为tIdx的普通索引

```sql
create index tIdx on t(id);
```

在t表中id字段创建名为tIdx的唯一索引

```sql
create unique index tIdx on t(id);
```

在t表中id字段创建名为tIdx的单列索引

```sql
create index tIdx on t(id);
```

在t表中id和name字段创建名为tIdx的组合索引

```sql
create index tIdx on t(id, name);
```

### 三、删除索引

##### 1.使用alter table删除索引

```sql
alter table t1 drop index id;
```

##### 2.使用drop index语句删除索引

删除上一章创建t2的索引，索引名为UniqIdx。

```sql
drop index UniqIdx on t2;
```