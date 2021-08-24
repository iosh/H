---
title: MySQL-必知必会
date: 2021-08-19 19:40:51
tags:
---

MySQL 必知必会 读书笔记

<!-- more -->

## 数据库三大范式

1. 要求属性的`原子性`
   最基本的范式, 要求数据库中所有字段的值是不可分解的原子值
2. 对记录的`唯一性`, 要求记录具有唯一标识, 不能有依赖
   在第一条的基础上,要求记录必须有唯一标识, 如果是单一主键, 则符合第二范式
3. 要求非主键字段和主键字段不能产生传递依赖
   在第二条的基础上要求试题中的属性不能是其他实体中的非主属性, 因为这样会造成数据冗余, 即属性不依赖于其他非主属性. 第三范式要求确保数据宝忠的每一列数据都和主键有直接相关联不能间接关联

## 名词

- 数据库(database) 保存有组织的数据的容器, 一般是一个或多个表
- 表(table) 一种结构化的文件, 用于储存某种特定类型的数据.
- 模式(schema) 关于数据库和表的布局的特性的信息
- 数据类型(datatype) 允许的数据的类型, 表中每一列都有相对应的类型, 限制该列中储存的数据
- 列(column) 表中的一个字段, 所有表都是由一个或者多个列组成的
- 行(row) 表中的数据是按照行来进行保存的
- 主键(primary key) 其值能够唯一区分表中每一行

## SQL

SQL 是一种结构化的查询语言, SQL 是专门用来与数据库通信的语言

- SQL 是几乎所有 DBMS 都支持的
- 看上去简单, 实际上很强力的语言,可以进行非常复杂和高级的数据库操作

## 基础使用

```sql
# 显示数据库
show databases;

# 显示表
show table;

# 显示表中的列
show columns from tablename;

# 显示广泛的服务器状态
show status;

# 显示可以用来授予用户的安全权限
show grants;

# 显示错误和警告
show errors;
show warnings;

# 显示 show 命令帮助
help show;
```

## 数据查查询

查询数据, 支持 \* 匹配所有, 支持多列.

```sql
select column_name from table_name;
```

使用 order by 字句可以对数据进行排列, 支持多个列, 按照顺序依次进行排序, 默认进行升序(A-Z), 使用 desc 关键字进行降序排序. 如果要对多列数据进行降序排列, 每列的名字前都需要加入 desc 关键字.

```sql
select * from table_name order by column_name;
```

使用 where 字句可以进行数据过滤. 支持常见运算符 = > < != >= 等, between 可以指定两个值之间

```sql
select * from table_name where id = 1;
```

mysql 支持多个 where 并且支持 and or 逻辑操作. 默认 and 操作符优先级更高, 可以通过添加 () 的方式来改变优先级.

in 操作符支持一个逗号分隔的列表

```sql
select * from table_name  where id =1; and name = 'max' and age in (19, 20, 99) order by age;
```

not 用于否定后面的操作

```sql
select * from table_name where id not in(1,2,3,4);
```

like 操作符

% 匹配任意数量任意字符

```sql
# 配合 % 可以匹配人和字母开头中间包括 j 人和字母结尾的用户名;
select * from table_name where username like '%j%';
```

\_ 匹配单一数量任意字符

通配符搜索一般比其他搜索时间花费更长, 几个优化建议, 不要过度使用通配符, 不要在搜索最开始使用通配符, 注意通配符位置

正则表达式搜索

```sql
select * from table_name where name REGXP 'max';
```

和 like 蕾西, REGXP 后面紧跟正则表达式

拼接字段

```sql
select Concat(RTrim(username), '(', age, ')') as t from user order by username;
```

`Concat` 拼接串, 可以把几列数据拼接为一列返回, `RTrim` 可以三处右侧多余空格来整理数据, `as` 可以设置数据别名.

并且字段可以进行 + - \* / 操作

常见 sql 函数, sql 函数支持程度不同, 各个数据库不一致.

`Upper` 转换为大写. `Length` 返回串的长度, `Locate` 找出串的一个子串, `Lower` 将串转换为小写, `LTirm` 去掉左边的空格, `SubString` 返回子串的字符

常见日期和时间处理函数

`AddDate` 增加一个日期, `AddTime` 增加一个时间 `CurDate` 返回当前时间 `CurTime` 返回当前时间 `Date` 返回日期时间的日期部分 `DateDiff` 计算两个时间差 `Date_Add` 高度灵活的日期运算函数, `Date_Format` 返回一个格式化的日期或时间串 `Day` 返回一个日期的天数部分 `DayOfWeek` 返回一个日期对应的星期几 `Hour` 返回一个时间的小时部分 `Minute` `Month` `Now` `Second` `Time` `Year` 返回时间的相应部分

汇总数据, 并不实际检索,常见有获取表中的行数, 获得表中行组之和, 找出最大值最小值平均值

`AVG` 返回某列的平均值 `COUNT` 返回某列的行数 `MAX` 返回某列的最大值 `MIN` 返回某列的最小值 `SUM` 返回某列之和

数据分组

使用 `group by` 子句可以对数据进行分组.

```sql
select id, count(*) as num_prods from products group by id;
```

可以得到根据 id 分组的数据

可以使用 `having` 对分组后的数据进行过滤

子查询

```sql
select id from users where username in (select username in members where age > 18);
```

使用 `insert` 插入数据

```sql
insert into users(id, username, age) values(null, 'max', 18);
```

使用 `update` 更新数据

```sql
update users set age = 28 where username = 'max';
```

使用 `delete` 删除数据

```sql
delete from users where username = 'max';
```


## 表操作

使用 `create table` 来创建表

```sql
create table users {
   id int not null auto_increment;
   username varchar(50) not null;
   age int not null;
   primary key(id);
}
```

每个表都应该有一个唯一的主键, 可以是单列也可以是多列组合, 形成唯一主键

使用 `drop table tablename` 来删除整个表,

使用 `rename table tablename to tablename` 重命名一个表, 可以指定多个表名进行多个重命名


## 视图

视图是虚拟的表. 视图只包含使用时动态检索的数据

视图常用方法, 重用 sql 语句, 简化复杂 sql 操作, 使用表的部分组成, 更改格式和表示等.

视图限制和规则, 命名必须唯一, 视图没有数量限制, 创建视图需要有权限, 视图可以嵌套, order by 可以用于视图中, 视图不能索引,

使用 `create view` 来创建视图.


