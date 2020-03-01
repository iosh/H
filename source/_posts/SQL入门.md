---
title: SQL入门
date: 2020-02-29 10:59:21
tags: SQL
---

SQL 学习

<!-- more -->

目标： 学习并会使用使用 SQL 为查询语言的数据库`使用`

数据库(datebase)是一种以某种有组织的方式储存的数据集合。

表: 表在数据库领域中,是一种结构化的文件,可以用来储存特定类型的数据,核心在于储存在表中的数据是`同一种`数据,同一种是指关系紧密相连的,例如顾客消费清单, 就不应该和库存清单储存在一起,如果储存在一起就会造成检索和访问更新等一系列困难,对于这种数据应该放在连个表中.

表具有一些特性,例如在创建表的时候会使用 SQL 描述表中的数据和数据类型,这种描述表的信息就是模式(schema)

列: 列是表中的一个字段,所有的表都是由一个或者多个列组成的,每一列都有对应的数据类型(datatype), 列的类型限制了当前列所能储存的信息. 不用的数据库对基础类型大多一致,但是不同的数据库会有一些独有的高级类型,需要具体区分.

行: 行是列的一部分, 表中的数据都是按照行来储存的

主键: primary id 是一行数据中的一个值,其值能够唯一标识表中的每一行, 主键用来表示一个特定的行,如果没有主键那么更新删除和修改特定的主键就会变得极为困难,因为不能保证操作只涉及相关的行.

表中的任何列都可以作为主键, 只要他满足下列条件:

- 值是唯一的,即任意两行的主键都不相同
- 每一行都必须拥有一个主键并且不能为 NULL
- 主键的值不允许修改和更新
- 主键的值不能重用, 被删除之后,被删除的主键也不能赋值给之后的新值

主键通常定义在表的一列上,但并不是必须这么做,`也可以一起使用多列作为主键`,在使用多列作为主键时,也必须符合上面的条件.

SQL 只有很少的词, 设计 SQL 的目的就是为了很好的完成一项任务,提供一种从数据库中读写数据的简单有效的方法.

# 数据检索

## SELECT 语句

为了使用 SELECT 语句, 必须至少给出两条信息 : `想选择什么` 以及 `从哪里选择`

### 检索单列

```sql
SELECT name FROM tableName
```

SELECT 语句将返回表中所有行,数据没有过滤,也没有排序.

多条 SQL 语句必须用分号结尾, 单条语句根据具体数据库实现不同有的需要有的不需要,如果都加上也没问题.

SQL 语句不区分大小写所以 SELECT 和 select 是相同的

### 检索多列

```sql
SELECT lineName,  lineName1 FROM tablename
```

### 检索所有的列

```sql
SELECT * FROM tableName
```

如果给你定一个通配符,则返回表中所有列,列的顺序一般是列在表中出现的物理顺序,但并不总是如此

### 检索不同的值

```sql
SELECT DISTINCT lineName FROM tableName
```

distinct 用于筛选表中不同的值,例如从都是 1 1 1 1 2 1 1 1 中筛选出 2 的值.

注意 distinct 作用于其后面的所有的查询列,而不是仅为其后面的一个.

### 限制结果

SELECT 语句返回指定表中的所有匹配行,很可能是每一行

```sql
SELECT lineName FROM tableName LIMIT 5;
```

使用 LIMIT 来返回前五条数据

```sql
SELECT lineName FROM tableName LIMIT 5 OFFSET 5
```

limet 5 offset 5 指示 MySQL 等数据库返回从第五行起(包含)的五条数据

注意被检索的行也是从 0 开始,而不是从 1 开始计数的.

MySQL MariaDB 数据库支持简写: limit 5, 5

### 注释

```sql
SELECT lineName FROM tableName; -- 这是注释
```

注释使用 -- (两个连字符)嵌在行内, -- 之后的文本就是注释

```sql

/*
多行注释
*/
SELECT lineName FROM tableName;
```

## 排序检索数据

### 排序数据

关系数据库理论认为,如果不明确规定排序顺序, 则不应该既定检索出的数据的顺序有任何意义.

字距 clause, SQL 语句由字句组成,有些字句是必须的,有些则是可选的,一个字句通常由一个关键字加上所提供的数据组成.例如 SELECT 和 FROM 子句

为了明确地使用 SELECT 检索出排序的数据,可以使用 ORDER BY 子句, ORDER BY 字句取一个或者多个列的名字,据此对数据进行排序

```sql
SELECT lineName FROM tableName ORDER BY lineName
```

### 按多个列排序

```sql
SELECT lineName1, lineName2, lineName3 from tableName ORDER BY lineName1 lineName2;
```

这里获取到的数据会先按照 lineName1 排序, 之后再按照 lineName2 进行排序. 并且仅在 lineName1 有多行相同数据的时候才会继续按照 lineName2 进行排序.

### 按位置排列

```sql
SELECT lineName1, lineName2, lineName3 from tableName ORDER BY 2,3
```

使用数字来进行排序指定是是表中相对的位置,而不是别名

### 指定排序方向

数据裴旭不限于升序(从 A-Z),这只是默认的排序顺序, 还可以使用 ORDER BY 子句进行降序 (Z-A) 排序, 为了进行降序排序, 必须指定 DESC 关键字

```sql
SELECT lineName, lineName1 FROM tableName ORDER BY lineName DESC
```

## 数据过滤

### 使用 WHERE 子句

数据库一般包含大量的数据, 很少需要检索数据表中的所有行, 通常只会根据特定操作或者报告的需要提取数据的子集, 只检索所需数据需要的指定搜索,被称为过滤搜索

```sql
SELECT lineName FROM tableName WHERE lineName = 'test'
```

这里只筛选了 lineName 的值等于 test 的数据

### WHERE 子句操作符

| 操作符  | 说明               |
| ------- | ------------------ |
| =       | 等于               |
| <>      | 不等于             |
| !=      | 不等于             |
| <       | 小于               |
| !<      | 不小于             |
| >       | 大于               |
| >=      | 大于等于           |
| !>      | 不大于             |
| BETWEEN | 在指定的两个值之间 |
| IS NULL | 为 NULL            |

有些操作符是冗余的 例如<> 和 != !< 与>=.并且并非所以数据库都支持这些操作符,具体需要看对应文档.

```sql

SELECT lineName, lineName FROM tableName WHERE lineName < 100;
```

### 不匹配检查

```sql

SELECT lineName, lineName1 FROM tableName WHERE lineName != 'test'
```

### 范围检查值

```sql
SELECT lineName, lineName fron tableName WHERE lineName BETWEEN 1 AND 10
```

### 空值检查

在创建表是,可以指定其中的列嫩否不包含值,在一个列不包含值是,称其包含空值 NULL

```sql
SELECT lineName, lineName1 FROM tableName WHERE lineName IS NULL
```

## 高级数据过滤

使用 WHERE 子句建立功能更强,更高级的搜索条件

### 组合 WHERE 子句

SQL 允许给出多个 WHRE 子句,这些子句有两种使用方式, 即使用 AND 或 OR 子句的方式使用

操作符 operator
用来连接或者改变 WHERE 子句中的子句的关键字,也称为逻辑操作符

### AND 操作符

```sql
SELECT lineName, lineName1, lineName2 FROM tableName WHERE lineName='test' and lineName <  100
```

### OR 操作符

OR 操作符和 AND 操作符正好相反

```sql
SELECT lineName, lineName1 FROM tableName WHERE lineName='test' or lineName1 <10
```

### 求值顺序

SQL 在处理 OR 操作符之前, 会优先处理 AND 操作符,可以通过添加 () 的方式来进行明确分组

```sql
SELECT lineName, lineName1,lineName2 FROM tableName WHERE (lineName = 'test' or lineName1 >10) and lineName2 > 1000
```

### IN 操作符

IN 操作符用来指定条件范围, 范围中的每个条件都可以进行匹配, IN 取一组由逗号,括在圆括号中的合法值

```SQL
SELECT lineName, lineName1 FROM tableName WHERE lineName in (1, 2) ORDER BY lineName
```

上面的语句使用 OR 的话

```sql
SELECT lineName , lineName1, FROM tableName WHERE lineName = 1 or lineName =2
```

使用 IN 操作符,其优点如下:

- 再有很多合法选项的时候, IN 操作符的语法更清楚,更直观
- 在与其他 AND 和 OR 操作符组合使用 IN 的时候,求值顺序更容易管理
- IN 操作符一般会比一组 OR 操作符执行更快
- IN 的最大优点是可以包含其他 SELECT 语句, 可以更动态的建立 WHERE 子句

### NOT 操作符

WHERE 子句中的 NOT 操作符有且只有一个功能, 那就是否定其后面所跟的任何条件, 因为 NOT 从不单独使用,所以他的语法与其他操作符不同, NOT 关键字可以用在要过滤的前列,而不是之后.

```sql
SELECT lineName, lineName1 FROM tableName WHRER NOT lineName = 'test' ORDER BY lineName1
```

## 用通配符进行过滤

### LIKE 操作符

通配符本身实际上是 SQL 的 WHERE 子句中有特殊函数以的字符, SQL 支持集中通配符, 为了在搜索子句中使用通配符, 必须使用 LIKE 操作符

谓词 predicate
操作符何时不是操作符, 当他做作为谓词时, 从技术上来讲, LIKE 是谓词而不是操作符

### 百分号 % 通配符

最长使用的通配符是百分号, 在搜索串中, 百分号标识任何字符出现任意次数,例如为了找到 Fish 开头的产品,可以使用以下 SELECT 语句:

```sql
SELECT lineNeme, lineName1 FROM tableName WHERE lineName LIKE 'Fish$'
```

通配符 % 不会匹配 NULL

### 下划线(\_)通配符

下划线的用途与%一样,但是他只匹配一个字符,而不是多个字符

```sql
SELECT lineName, lineName1 FROM tableName WHERE lineName LIKE  '_ish'
```

### 方括号 [] 通配符

方括号通配符用来指定一个子集, 必须匹配指定位置(通配符的位置)的一个字符
例如要找出一个以 M 或者 J 开头的单词可以进行如下查询

```sql
SELECT lineName, lineName1 FROM tableName WHERE lineName LINE '[MJ]%'
```

### 通配符使用技巧

SQL 通配符通常很有用,但是这种功能是有代价的,即通配符搜索会比普通搜索耗费更长的处理时间,以下是一些技巧

- 不要过度使用通配符, 如果其他操作符能实现相同的目的, 应该使用其他操作符
- 在确实需要使用通配符时, 尽量不要把他们用在搜索模式开始,搜索起来是最慢的
- 仔细注意通配符的位置, 如果放错地方, 可能不会返回想要的数据

## 创建计算字段

### 计算字段

储存在数据库表中的数据一般不是应用程序所需要的格式,下面举个例子:

- 需要显示公司名,同时还需要显示公司的地址,但是这两个信息储存在不同的表中
- 城市, 州和邮政编码储存在不同的列中,但是邮件标签打印程序需要把他们作为一个由恰当格式的字段检索出来
- 列数据是大小写混合的,但是报表程序需要把所有数据按大写表示出来
- 物品订单表春村的价格和数量, 不储存每个物品的总价格,但为了打印发票需要计算总价格
- 需要根据表数据进行诸如总数,平均数的计算

### 拼接字段

拼接 concatenate
将值连接到一起构成单个值

最简单的办法是吧俩个值拼接起来, 在 SQL 中的 SELECT 语句中,可以使用一个特殊的操作符拼接两个列, 根据所使用的数据库不同, 操作可以使用 + 或者 || 标识, 在 MySQL 和 MariaDB 中,必须使用特殊的标识.

在 MySQL 和 mariaDB 中徐亚使用 Concat

```sql
SELECT Concat(lineName, '(', lineName1, ')') FROM tableName
```

### 使用别名

使用 SELECT 语句可以很好的拼接字段, 但是这个计算出来的列并没有名称, 他只是一个值,所以没办法引用他, 为了解决这个问题, SQL 支持列别名, 别名 alias , 别名用 AS 关键字赋值.

```sql
SELECT RTRIM(lineName ) + '(' + RTRIM(lineName1) +')' as testName FROM tableName
```

在许多数据库中 AS 通常是可选的, 但是最好使用它,这被视为最佳实践

### 执行算数计算

计算字段的另一个常见用途是对检索出的数据进行算数计算

```sql
SELECT lineName, lineName1 * lineName2 AS testName FROM tableName
```

SQL 中可以使用的算数操作符

| 操作符 | 说明 |
| ------ | ---- |
| +      | 加   |
| -      | 减   |
| \*     | 乘   |
| /      | 除   |

## 使用函数处理数据

### 函数

大多数函数都是不可移植的,但是如果不使用函数,编写某些应用程序代码会很艰难

### 使用函数

大多数 SQL 实现支持以下类型的函数

- 用于处理文本字符串, 如删除或者填充,转换值为大写或者小写的文本函数
- 用于在数值数据上进行算数操作, 如范湖绝对值,进行代数运算的数值函数
- 用于处理日期和时间值并从这些值中提取特定成分,例如返回两个时间之差,的日期和时间函数
- 返回数据库正在使用的特殊信息的系统函数

### 处理文本函数

```sql
SELECT UPPER(lineName) from tableName
```

常用的文本处理函数

| 函数                                         | 说明                     |
| -------------------------------------------- | ------------------------ |
| LEFT() (或使用子字符串函数)                  | 返回字符串左边的字符     |
| LENGTH() (也可以使用 DATALENGTH()或者 LEN()) | 返回字符串的长度         |
| LOWER()                                      | 将返回的字符串转换为小写 |
| LERIM()                                      | 去掉字符串左边的空格     |
| RIGHT()(或使用子字符串函数)                  | 返回字符串左边的字符串   |
| RTRIM()                                      | 去掉字符串右边的空格     |
| SOUNDEX()                                    | 返回字符串的 SOUNDEX 值  |
| UPPER()                                      | 将字符串转换为大写       |

### 日期和时间处理函数

日期和时间采用响应的数据类型储存在表中,每种数据库都有自己的特殊格式,日期和时间以特殊的格式储存,以便可以快速和有效的排序和过滤数据,并且节省物理储存空间

数据库提供的函数都不一样

### 数值处理函数

| 函数 | 说明               |
| ---- | ------------------ |
| ABS  | 返回一个数的绝对值 |
| COS  | 返回一个角度的余弦 |
| EXP  | 返回一个数的指数值 |
| PI   | 返回圆周率         |
| SIN  | 返回一个角度的正弦 |
| SQRT | 返回一个数的平方根 |
| TAN  | 返回一个角度的正切 |

## 汇总数据

### 聚集函数

经常需要汇总数据而不用刚把他们实际检索出来,为此 SQL 提供了专门的函数,使用这些函数,SQL 查询可以用于检索数据,以便分析和报表生成, 例如这种类型的检索列子有:

- 确定表中的行数(或者满足某个条件或者包含某个特定值的行数)
- 获得表中某些行的和
- 找出列表(或者某些特定行)中的最大值,最小值,平均值等

上面的例子都只需要会中表中的一些数据, 而并不需要实际数据本身,因此不必查询说有数据.

为了应对这种情况,SQL 给出了五个聚集函数,这些函数能进行上述检索.

| 函数  | 说明             |
| ----- | ---------------- |
| AVG   | 返回某列的平均值 |
| COUNT | 返回某列的行数   |
| NAX   | 返回某列的最大值 |
| MIN   | 返回某列的最小值 |
| SUM   | 返回某列的和     |

## 分组数据

### 创建分组

分组使用 SELECT 语句的 GROUP BY 子句建立

```sql
SELECT lineName, COUNT(lineName1) FROM tableName GROUP BY lineName
```

再使用 GROUP BY 子句前, 需要知道一些重要的规定:

- GROUP BY 子句可以包含任意数目的列, 因而可以对分组进行嵌套,更细致的进行数据分组
- 如果在 GROUP BY 子句中嵌套了分组, 数据将在最后指定的分组上进行归总,换句话说,在建立分组时,指定的所有列都是一起计算
- GROUP BY 子句中列出的数据都必须是检索列或有效的表达式(但不能是聚集函数), 如果在 SELECT 中使用过表达式,则必须在 GROUP BY 子句中使用相同的表达式,不能使用个别名
- 大多数 SQL 实现不允许 GROUP BY 列带有长度可变的互数据类型]
- 除聚集计算语句外, SELECT 语句中每一列都必须在 GROUP BY 子句中给出
- 如果分组列中包含具有 NULL 值, 则将 NULL 作为一个分组返回, 如果有多个 NULL 将会被视为一组
- GROUP BY 子句必须出现在 WHERE 子句之后, ORDER BY 子句之前

### 过滤分组

除了能够使用 GROUP BY 分组数据之外, SQL 还允许过滤分组, 规定包括哪些分组, 排除哪些分组.

SQL 提供了 HAVING 语句, WHERE 过滤行, HAVING 过滤分组.并且 HAVING 支持 WHERE 的所有操作.

HAVING 和 WHERE 的差别:

简单的理解就是 WHERE 在数据分组前进行过滤, 而 HAVING 在数据分组之后对分组进行过滤.

```sql
SELECT lineName, COUNT(lineName1) FROM tableName WHERE lineName > 100 GROUP BY lineName HAVING COUNT(lineName1) < 10
```

### SELECT 子句顺序

| 子句     | 说明                 | 是否必须使用           |
| -------- | -------------------- | ---------------------- |
| SELECT   | 要返回的列或者表达式 | 是                     |
| FROM     | 从中检索数据的表     | 仅在从表选择数据时使用 |
| WHERE    | 行级过滤             | 否                     |
| GROUP BY | 分组说明             | 仅在安祖计算聚集时使用 |
| HAVING   | 组级过滤             | 否                     |
| ORDER BY | 输出排序序列         | 否                     |

## 使用子查询

### 子查询

子查询就是嵌套在其他查询中的查询

### 利用子查询进行过滤

订单储存储存在两个表中, 每个订单包含编号,客户 ID, 订单日期, 各订单的物品储存在相关的另一个表中, 一个表储存顾客 ID, 另一个表储存实际信息

现在需要列出订购物品 RGAN01 的所有顾客,可以按照如下方式进行检索:

- 检索包含物品 RGAN01 的所有订单的编号
- 检索具有前一个步骤列出的订单编号的所有顾客 ID
- 检索迁移步骤返回的所有顾客 ID 的顾客信息

上述的步骤都可以单独作为一个查询来执行, 可以吧一条 SELECT 语句返回的结果用于另一条 SELECT 语句的 WHERE 子句

```sql
SELECT  cust_id  FROM Orders WHERE order_num IN (SELECT  order_num FROM OrderItems WHERE  prod_id = 'RGAN01')
```

在 SELECT 语句中, 子查询总是从内向外处理, 在处理上面的 SELECT 语句时, 实际上数据库执行了两个操作

SELECT order_num FROM OrderItems WHERE prod_id = 'RGAN01'

得到结果后
SELECT cust_id FROM Orders WHERE order_num IN 上一条语句得到的结果

注意 作为 子查询的 SELECT 只能查询单个列, 检索多个列将会报错.

## 联结表

### 联结

SQL 最强大的功能之一就是在数据查询执行中联结 join 表,联结是利用 SQL 的 SELECT 能执行的最重要的操作

### 关系表

例如:

有一个包含产品目录的数据库表, 其中没类物品占一行, 对于每一种物品,要储存的信息包括产品描述, 价格, 以及生产该产品的供应商

现在有同一供应商生产的多种物品,那么在何处储存供应商,地址联系方式等供应商信息,将这些信息分开储存的理由:

- 同一供应商生产的每个产品, 其供应商的信息都是相同的,对于每个产品重复此信息即浪费时间又浪费储存空间
- 如果供应商信息发生变化, 例如供应商地址或者联系方式发生变化,只需要更新一次即可
- 如果有重复数据, 则很难保证每一次数据该数据的方式都相同,不一致的数据在报表中很难利用

### 为什么使用联结

将数据分解为多个表能更有效的储存,更方便的处理,并且可伸缩性更好, 但是这些好处是有代价的

如果数据储存在多个表中, 使用联结,就可以在一条 SELECT 语句中检索出数据, 联结是一种机制,用于在一条 SELECT 语句中关联表, 因此成为联结,使用特殊的语法可以联结多个表返回一组输出,联结在运行时关联表中正确的行

### 创建联结

```sql
SELECT lineName, lineName1, lineName2 FROM tableName, tabLeName1 WHERE tableName.vend_id = tableName1.vend_id
```

### 内联结

```sql
SELECT lineName, lineName1, lineName2 FROM tableName INNER JOIN tabLeName1 WHERE tableName.vend_id = tableName1.vend_id
```

通过 inner join 明确 指出联结的类型

### 联结多个表

SQL 不限制一条 select 语句中可以联结表的数目, 创建连接的基本规则也相同,首先列出所有表,然后定义表之间的关系

例如

```sql

SELECT prod_name. vend_name, prod_price, quanticty FROM OrderItems, Products, Vendors WHERE Products.vend_id = Vendors.vend_id AND OrderItems.prod_id = Products.prod_id
```

性能考虑:
数据库在运行关联指定的每个表,以处理连接,这种处理可能非常耗费资源, 因此应该注意,不要连接不必要的表,连接的表越多,性能下降的越厉害

## 创建高连接

### 使用表别名

```sql
SELECT lineName, lineName1 FROM tableName AS t, tableName1 AS d WHERE t.id = d.id
```

表别名只会在查询中使用, 与 列别名不同的是, 表别名不会返回到客户端

### 使用不同类型的联结

除了内联结还有其他三种联结 自联结, 自然联结, 外连接

### 自联结

使用表别名的主要原因是能够在一条 SELECT 语句中不止一次引用相同的表.例如:

加入要给 Jim Jones 统一公司的所有顾客发送一封邮件,这个查询要修先找出 Jim Jones 工作的公司, 然后找出在该公司工作的顾客

最普通的方法:

```sql
SELECT cust_id ,cust_name,cust_contact FROM Customers WHERE curs_name = (SELECT cust_name FROM Customers WHERE cust_contact = 'Jim Jones')
```

使用联结查询

```sql

SELECT c1.cust_id, c1.cust_name,c1.cust_contact FROM Customers AS c1, Customers AS c2 WHERE c1.cust_name c2.cust_name and c2.cust_name = 'Jim Jones'
```

### 自然联结

无论何时对表进行联结, 应该至少有一列不止出现在一个表中,标准的连接返回所有数据, 相同的数据甚至可能出现多次,自然联结排除出现多次,使每一列只返回一次.

自然联结要求你只选择那些唯一的值, 一般通过对一个表用通配符 select \* 而对其他的列使用明确地子集来完成

```sql
SELECT C.*, O.order_num, O.order_date, OI.prod_id, OI.quantity, OI.item_price FROM Customers AS C, Order AS O, OriderItem AS OI WHERE OI.order_num = O.order_num AND prod_id = 'RGAN01'
```

### 外连接

许多连接是将一个表的行与另一个表中的行向关联,但有时候需要包含没有关联的那些行,例如需要使用联结完成以下工作.

- 对每个顾客下的订单进行计数, 包括哪些至今尚未下订单的顾客
- 列出所有产品以及订购数量,包括没有人订购的产品
- 计算平均销售规模, 包括哪些至今尚未下单的顾客

下面 SELECT 语句给出了一个简单的内联结, 它检索所有顾客及其订单

```sql
SELECT Customers.cust_id , Orders.order_num FROM Customers INNER JOIN Orders ON CUstomers.cust_id = Orders.cust_id
```

/====外联结语法类似, 要检索包括没有订单顾客在内的所有顾客, 可如下进行:

```sql
SELECT Customers.cust_id , Orders.order_num FROM Customers LEFT OUTER JOIN Orders ON Customers.cust_id = Orders.cust_id
```

## 组合查询

### 组合查询

多数 SQL 查询 只包含从一个或者多个表中返回数据的单条 SELECT 语句, 但是 SQL 也允许执行多个查询(多条 SELECT 语句),并将结果作为一个查询结果集返回, 这种组合查询通常称为 并 union 后者符合查询 copund query

主要有爱你高中情况需要使用组合查询:

- 在一个查询中从不同的表返回结构数据
- 对一个表执行多个查询, 按照一个查询返回数据

多数情况下, 组合相同表的两个查询所完成的工作与具有多个 WHERE 子句条件的一个查询所完成的工作相同, 换句话说,任何具有多个 WHERE 子句的 SELECT 都可以作为一个组合查询

### 创建组合查询

可以使用 UNION 操作符来组合数条 SLQ 查询,利用 UNION 可以给出多条 SELECT 语句, 将他们的结果作为一个结果集返回

### 使用 UNION

使用 UNION 很简单, 所需要做的只是给出每条 SELECT 语句, 之间放入关键字 UNION

```sql
SELECT lineName , lineName1 FROM tableName UNION SELECT lineName3 FROM tableName2
```

### UNION 规则

UNION 使用起来需要注意几条规则:

- UNION 必须由链条或者两条 SELECT 语句组成, 语句之间关键字 UNION 分隔(因此, 如果组合四条 SELECT 语句, 将要使用三个 UNION)
- UNION 中的每个查询必须包含相同的列,表达式或聚集函数(不过,各个列必须要以相同的次序列出)
- 列数据类型必须兼容 : 类型不必完全相同, 但是必须是数据库可以隐含类型的转换

### 包含或取消重复的行

UNION 从查询结果集中个自动去除了重复的行,换句话说, 他的姓行为与一条 SELECT 语句中使用多个 WHERE 子句条件一样

### 对组合查询结果排序

SELECT 语句的输出用于 ORDER BY 子句排序, 在 UNION 组合查询时, 只能使用一条 ORDER BY 子句, 他必须位于左后一条 SELECT 语句之后, 对于结果集, 不存在用一种排序一部分,而又对另一种方式排序另一部分的情况,因此不允许使用多条 ORDER BY 子句

## 插入数据

## 数据插入

SQL 使用 INSERT 来将行插入或添加到数据表,插入有几种方式:

- 完成插入行
- 插入行的一部分
- 插入某些查询的结果

```sql
INSERT INTO tableName VALUES(
    'value',
    'value'
)
```

上面的值根据表中列的顺序依次列出值,这种语法虽然简单,但是极度依赖获得的次序信息,如果次序发生变化维护起来会很麻烦.

因此可以编写更加安全的(不过更加繁琐)的方法:

```sql
INSERT INTO tableName (
   id,
   testName
) VALUES(1, 'test')

```

无论使用哪种 INSERT 语法, VALUES 的数目必须正确, 如果不提供列名, 则必须给每个列表提供一个值, 如果提供列名, 则必须每个列一个值,否则会产生错误

### 插入部分行

使用 INSERT 提供名称的方法,可以只提供部分名称,从而只提供部分值

如果省略列, 那么必须满足下列条件:

- 该列允许定义 NULL 值, 无值或者空值
- 在表定义中给出默认值, 这表示如果不给出值,将使用默认值

### 插入检索出的的数据

INSERT 一般用来给表插入具有指定列值的行, INSERT 还存在另一种形式, 可以利用它将 SELECT 语句的结果插入表中, 这就是 INSERT SELECT 顾名思义, 它是由一条 INSERT 语句和一条 SELECT 语句组成的

```sql
INSERT INTO Customers(cust_id, cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country) SELECT cust_id, cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country FROM CustNew
```

同时 INSERT SELECT 中 SELECT 语句可以包含 WHERE 子句,以过滤插入的数据

### 从一个表赋值到另一个表

要将一个表的内容复制到一个全新的表(运行中创建的表), 那么可以使用 SELECT INTO 语句

与 INSERT SELECT 将数据添加到一个存在表不同, SELECT INTO 将数据复制到一个新表

INSERT SELECT 和 SELECT INTO 的重要区别是前者插入数据,而后者导出数据

```sql
SELECT  *  INTO CustCopy FROM Customers;
```

## 更新和删除数据

UPDATE 和 DELETE 语句使用

### 更细数据

更新修改表中的数据,可以使用 UPDATE 语句, 有两种使用 UPDATE

- 更新表中特定行
- 更新表中所有行

在使用 UDATE 的时候一定要仔细小心,因为稍不注意就会更新表中所有行,

基本的 UPDATE 语句由三部分组成,分别是:

- 要更新的表
- 列名和他们的新值
- 确定要更新哪些过滤条件

```sql
UPDATE Customers SET cust_emaile= 'xx@xc.com' WHERE cust_id = '100000'
```

### 删除数据

使用 DELETE 语句, 有两种 DELETE 的方式:

- 从表中删除特定的行
- 从表中删除所有的行

```sql
DELETE lineName FROM tableName WHERE id= '1'
```

DELETE 语句从表中删除行, 甚至是删除表中所有行, 但是 DELETE 并不删除表本身

### 更新和删除的指导原则

使用 UPDATE 和 DELETE 语句都有 WHERE 子句, 这样做可以避免删除所有行

下面是一些使用 UPDATE 和 DELETE 时所遵循的重要原则

- 除非确实打算更新和删除每一行, 否则绝对不要使用不带 WHERE 子句的 UPDATE 或者 DELETE 语句
- 保证每个表都有主键, 尽可能的像 WHERE 子句那样使用它
- 在 UPDATE 或 DELETE 语句使用 WHERE 子句之前, 应该先用 SELECT 进行测试, 保证他过滤的是正确的记录, 以防编写的 WHERE 子句不正确
- 使用强制试试引用完整性的数据库, 这样数据库将不允许删除其数据与其他表关联的行

如果 数据库 没有撤销功能,应该非常小心的使用 UPDATE 和 DELETE

## 创建和操纵表

### 创建表

创建表一般哟两种方法:

- 多数数据库都具有交互式创建和管理数据库工具
- 是用 SQL 语句进行创建

### 表创建基础

利用 CREATE TABLE 创建表, 必须给出下列信息:

- 新的表名称, 在关键字 CREATE TABLE 之后给出
- 表列的名称和定义, 用逗号分隔
- 有的数据库还需要指定表的位置

```SQL
CREATE TABLE Products(
    prod_id CHAR(10) NOT NULL,
    vend_id CHAR(10) NOT NULL,
)
```

### 使用 NULL 值

NULL 值就是没有值或者缺值, 允许 NULL 值的列也允许在插入时不给出该列的值, 不允许 NULL 值的列不接受没有列值的, 换句话说,在插入和更新时, 该列必须由值

每个表列要么是 NULL 列, 要么是 NOT NULL 列, 这种状态在创建时由表的定义规定

### 更新表

使用 ALTER TABLE 语句可以更新数据库, 数据库支持差别比较大, 以下是使用 ALTEAR TABLE 时需要考虑的事情.

- 理想情况下, 不要在表中包含数据时对其进行更新,应该在标的设计过程中充分考虑未来可能的需求,避免今后对表的结构做大的改动
- 所有数据库都允许给现有的表增加列,不过对所有增加的数据类型有所限制
- 许多数据库不允许删除或更改表中的列
- 许多数据库允许重命名表中的列
- 许多数据库限制对已经天佑数据的列进行更改, 对未天有数据的列几乎没有限制

### 删除表中列

```sql
ALTER TABLE Products DROP COLUMN vend_id;
```

复杂的表结构更改一般需要手动删除过程, 它涉及以下步骤:

- 用新的列布局创建一个新表
- 使用 INSERT SELECT 语句从旧表中复制数据到新表
- 检验包含所需数据的新表
- 重命名旧表 如有必要删除旧表
- 用旧表原来的名字重命名新表
- 根据需要重新创建触发器, 储存过程, 索引和外键

### 删除表

删除表使用 DROP TABLE 语句

```sql
DROP TABLE Products
```

注意, 删除没有确认也没有取消,慎重操作

## 使用视图

### 视图

视图是虚拟的表, 与包含数据的表不一样, 视图只包含使用时动态检索数据的查询.

例如可以将一个非常复杂的查询包装成一个视图虚拟表,从而简化查询.

### 为什么使用视图

下面是一些常见的视图应用:

- 重用 SQL 语句
- 简化复杂的 SQL 操作, 在编写查询后,可以方便地重用它而不必知道其具体查询细节
- 使用表的一部分而不是整个表
- 保护数据, 可以授予用户访问表的特定部分的权限,而不是整个表的访问权限
- 更改数据格式和标识,视图可返回与底层表的标识和格式不同的数据

重要的是,要知道视图仅仅是用来查看储存在别处数据的一种设施, 视图本身并不包含数据,因此返回的数据是从其他表中检索出来的, 在添加或更改这些表中的数据是,视图将返回改变过的数据

性能问题, 因为视图不包含数据,所以每次使用视图是,都必须处理查询执行需要的所有检索,乳沟使用了多个联结和过滤创建了非常复杂的视图或者嵌套视图, 那么性能可能会下降的很厉害,因此在不是了使用大量视图应用之前,应该进行测试

### 视图的规则和限制

视图有一些限制,具体限制需要看具体数据库的文档,下面是一些常见的规则和限制:

- 与表名一样, 视图名必须唯一
- 对于可以创建的视图数目没有限制
- 创建视图, 必须有足够的的访问权限
- 视图可以嵌套, 即可以利用其它视图中检索的数据来构造视图, 所允许嵌套的层数在不同的数据库中并不相同.
- 许多数据库不允许在视图中使用 ORDER BY 子句
- 有些数据库要求对返货的多有列进行命名,如果列是计算字段,则需要使用别名
- 视图不能索引,也不能有关联的触发器或者默认值
- 有些数据吧视图作为只读查询,这表示可以从视图检索数据,但不能向其写入数据
- 有些数据库允许建立这样的视图, 他不能进行导致行不在属于视图的插入或者更新

### 创建视图

视图使用 CREATE VIEW 语句进行创建,并且只能用于创建不存在的视图

### 利用视图简化复杂的联结

最常见的视图应用是隐藏复杂的 SQL 这通常涉及到联结, 例如

```sql
CREATE VIEW ProductCustomers AS SELECT cust_name, cust_contact, prod_id FROM Customers, Orders, OrderItems WHERE Customers.cust_id = Orders.cust_id AND OrderItems.order_num = Orders.order_num
```

 上面就创建了一个 ProductCustomers 的视图,它联结三个表,返回已经丁欧了人异常品的所有顾客的列表, 如果执行 SELECT \* FROM ProductCustomers 将列出订购了任意产品的顾客

## 使用过程储存

### 储存过程

在实际使用过程中经常会有一些复杂的操作需要多条语句才能完成, 例如:

- 为了处理订单, 必须核对以保存库存中有相应的物品
- 如果物品有库存,需要预定, 不在出售给别人, 并且减少物品数据以反映正确的库存量
- 库存中没有的物品需要订购,这需要与供应商进行某种交互
- 关于哪些物品入库和哪些物品退订,需要通知相应的顾客

### 为什么使用储存过程

储存过程的一些主要理由:

- 通过吧处理封装在一个易用的单元中,可以简化复杂的操作
- 由于不要求反复建立一系列处理步骤, 因而保证了数据的一致性,如果所有开发人员和应用程序都是用同一储存过程, 则所使用的代码都相同.这一点的延伸就是防止错误,需要执行的步骤越多,出错的可能性就越大,防止错误保证了数据的一致性
- 简化对于变动的管理, 如果表名, 列名或者业务逻辑有变化,那么只需要更改储存过程的代码,使用它的人甚至不需要知道这些变化,这一点的延伸性就是安全性,通过储存过程限制对基础数据的访问,减少了失误的机会
- 因为储存过程通常以编译过的星系储存,所以数据库处理命令所需额工作量少,提高了性能
- 存在一些职能用在单个请求的 SQL 元素和特性, 储存过程可以使用它们来编写功能更强的灵活的代码

所以储存过程有三个主要的好处: 安全 简单 高性能, 但他也有一些缺点:

- 不同的数据库储存过程的语法有所不同,事实上编写可移植的过程储存几乎是不可能的,不过储存过程的自我调用可以相对保持可移植, 因此如果要移植到别的数据库,至少客户端应用代码不需要变动.
- 一般来说, 编写储存过程比编写基本 SQL 语句复杂,需要更高的性能, 更丰富的经验,因此许多数据库管理员吧限制储存过程的创建视为一种安全措施(主要受上一条的影响)

### 执行储存过程

使用 EXECUTE 接收储存过程需要传递给他的任何参数

```sql
EXECUTE AddNewProduct('JTS01','Styffed Eiffel Tower', 6.49)

```

这里执行了一个名为 AddNewProduct 的储存过程, 将一个新的产品添加到 Product 表中, 几个参数匹配储存过程预期的几个变量, 此储存过程将新行添加到 Product 表中,并将传入的属性附加给响应的列.

上面就是执行储存过程的基本形式, 对于具体的数据库, 可能包括以下的执行选择:

- 参数可选, 具有不提供参数时的默认值
- 不按次序给出参数, 以 "参数=值" 的方式给出参数值
- 输出参数, 允许储存过程在执行的应用程序更细所用的参数
- 使用 SELECT 语句检索数据
- 返回代码, 允许储存过程返回一个值到正在执行的应用程序

### 创建储存过程

```sql
CREATE PROCEDURE MailingListCount (
    ListCount OUT INTEGER
)
IS v_rows INTEGER;
BEGIN
  SELECT  COUNT(*) INTO v_rows FROM Customers WHERE NOT cust_email IS NULL;
  ListCount := v_rows;
END;
```

上面的例子中过程储存有一个 名为 ListCount 的参数, 这个参数从过程储存返回一个值而不是传递一个值给过程储存, 关键字 OUT 表示这个行为,

## 管理事务处理

### 事务处理

使用事务处理, 通过确保成批的 SQL 操作要么完全执行, 要么完全不执行, 来维护数据库的完整性

关系数据库吧数据库储存在多个表中, 使数据库更容易操作,维护和重用

添加数据都是一系列操作, 假设由于某种数据库股中昂, 这个过程无法完成, 就可能产生问题

对于解决这种问题, 就需要使用事务处理, 事务处理是一种机制, 用来管理必须成批执行的 SQL 操作, 保证数据库不会包含不完整的操作, 利用事务处理, 可以保证一组操作不会中途停止, 要么完全执行要么完全不执行, 乳沟没哟发生错误, 整组数据提交到数据表, 如果发生错误,则进行回退,将数据库恢复到某个已知安全的状态.

在使用事务处理时, 有几个反复出现的关键词,下面是关于事务处理需要知道的几个术语:

- 事务(transaction) 指一组 SQL 语句
- 回退(rollback) 指撤销指定 SQL 语句的过程
- 提交(commit) 指将为储存的 SQL 语句结果写入数据库表
- 保留点(savepoint) 指事务处理设置的临时占位符,可以对他发布退回

事务处理用来管理 INSERT UPDATE DELETE 语句, 不能回退 SELECT 也不能回退 CREATE 或者 DROP 操作.

### 控制事务处理

MySQL

```sql
START TRANSACTION;

```

### 使用 rollback

```sql
DELETE FROM tableName;
ROLLBACK;
```

上面执行了删除操作,然后使用 ROLLBACK 撤回了删除操作

### 使用 COMMIT

一般的 SQL 语句都是针对数据库表直接执行和编写的, 这就是所谓的隐式提交, 即提交或者写入的操作是自动的

在事务处理模块中, 提交不会隐式进行, 不过不同数据库的做法有所不同个, 有的 数据库按隐式提交处理事务端,有些则不会

### 使用保留点

使用简单的 ROLLBACK 和 COMMIT 语句,就可以写入或撤销真个事务, 但是,支队简单的事务才能这么做,复杂的事务可能需要部分提交或者回退

要支持回退部分事务, 必须在事务处理块中的何时位置放置占位符, 这样如果需要回退,可以回退到某个占位符

```sql
ROLLBACK TO delete1
```

## 使用游标

### 游标

SQL 检索操作返回一组称为结果集的行, 这组返回的行都是与 SQL 的语句向匹配的行, 简单的使用 SELECT 语句,没办法得到第一行下一行或者前世行,但是这是数据库的功能组成部分

有时候,需要在检索出来的行中进行前进或者后退多少航,这就是游标 cursor 是一个储存在数据库服务器上的数据库查询, 他不是一条 SELECT 语句, 而是被该语句检索出来的结果集, 在储存了游标之后,应用程序就可以根据需要滚动或者浏览其中数据

不同数据库支持不同的选项和特性, 下面是一些常见的选项和特性:

- 能够标记游标为制度, 使数据能够读取,但不能更新和删除
- 能控制可以执行的定向操作
- 能标记某些列为可编辑的, 耨写不可编辑
- 规定范围, 使游标对创建它的特定请求或对所有请求可访问
- 指示数据库对检索出的数据进行复制, 使数据在游标打开和访问期间不变化

### 使用游标

使用游标涉及几个明确的步骤:

- 在使用游标前,必须声明它,这个过程实际上没哟检索数据,它只是定义要使用的 SELECT 语句和游标选项
- 一旦声明, 就必须打开游标以供使用,这个过程用前面定义的 SELECT 语句把实际检索出来
- 对于天有数据的游标, 根据需要取出(检索)各行
- 在结束游标使用时, 必须关闭游标, 可能的话,释放游标(依赖于具体数据库实现)

### 创建游标

使用 DECLARE 语句创建游标, 这条语句在不同的数据库中该有所不同, DECLARE 命名游标, 并定义相应的 SELECT 语句, 根据需要带 WHERE 和其他子句.

```sql
DECLARE CustCursor CURSOR FOR SELECT * FROM Customers WHERE cust_email IS NULL
```

### 使用游标

使用 OPEN CURSOR 语句打开游标, 这条语句在大多数数据库中语法都相同

```sql
OPEN CURSOR CistCusor
```

之后就可以使用 FETCH 来指出要检索哪些行, 从何处检索他们以及将他们防止何处

## 高级 SQL 特性

### 约束

SQL 已经改进过很多个版本, 成为了非常完善和强大的语言, 许多强有力的特性给通用提供了高级数据处理技术,如约束

正确的进行关系数据库设计, 需要一种方法保证只在表中插入合法数据,虽然可以在插入新行时进行检查, 但是最好不要这样做

- 如果在客户端层面上实施数据库完整性规则, 则每个客户都被迫实施这些规则, 但是一定有客户不需要实施这些规则
- 在执行 UIPDATE 和 DELETE 操作时, 也必须实施这些规则
- 执行客户端检查是非常耗时的,而数据库进行这些检查会好很多

### 主键

如果没有主键, 要安全的 UPDATE 和 DELETE 特定航而不影响其他行会非常困难

表中任意列只要满足以下条件,都可以用于主键

- 任意两行的主键值都不相同
- 每行都具有一个主键值
- 包含主键值的列从不修改或更新
- 主键不能重用, 如果从表中删除一行,其主键值不能重新分配

```sql
CREATE TABLE Test (
    main_id INT NOT  NULL  PRIMARY KEY
)
```

### 外键

外键是表中的一列,其值必须列在另一个表的主键中, 外键是保证引用完整性的极为重要的一部分.

```sql
CREATE TABLE Order (
    cust_id INTEGER NOT  NULL PRIMARY KEY,
    other_id INTEGER NOT  NULL REFERENCES tableName(cust_id)
)
```

### 唯一约束

唯一约束用来保证一列或者一组列中的数据是唯一的, 他们类似于主键, 但存在以下重要区别

- 表可以包含多个唯一约束, 但是一个表只允许有一个主键
- 唯一约束可以包含 NULL 值
- 位于约束列可以修改和更新
- 唯一约束的值可以重复
- 与主键不一样, 唯一约束不能用来定义外键

### 检查约束

检查约束用来保证一列中的数据满足一组指定的条件:

- 检查最小或者最大值
- 检查范围
- 只允许特定值

### 索引

索引用来排序数据以加快搜索和排序操作的速度

主键数据总是排序的,这是数据库的工作, 因此 按照主键特定行总是一种快速有效的操作,但是搜索其他列中的值通常效率不高, 解决这个问题的办法就是建立索引, 可以在一个列或者多个列建立索引,使数据库保存其内容的一个排过序的列表, 在定义了索引后, 数据库可以使用它.

在创建索引之前,应该记住以下内容

- 索引改善检索操作的性能,但是降低了数据插入 修改和删除的性能
- 索引数据可能要占用大量的储存空间
- 并非所有数据都适合做索引, 取值不多的数据例如(州)不如具有更多可能值的数据(例如姓名),能通过索引得到的好处多
- 所以用于数据过滤和说句排序, 如果经常以某种特定顺序排序数据,那么该数据适合做索引
- 可以在索引中定义多个列

```sql
CREATE INDEX lineNameIndex ON tableName(lineName)
```

索引名称必须唯一, 索引名称在 INDEX 关键字之后, ON 用来指定被索引的表 之后是表名 表名括号内是列名

### 触发器

触发器是特殊的储存过程, 他在特定的数据发生活动时自动执行,触发器可以与特定表上的 INSERT UPDATE DELETE 操作相关联.

与过程储存不一样(储存过程只是简单的储存 SQL 语句),触发器与单个的表相关联

触发器的代码具有以下数据的访问权限:
- INSERT 操作中的所有新数据
- UPDATE 操作中的所有新数据和旧数据
- DELETE 操作中删除的数据


以下是一些触发器的常见用途:
- 保证数据一致, 例如在 INSTER 和 UPDATE 操作中将所有的名称转换为大写
- 基于某个表的变动在其他表上执行活动, 例如数据发生变化时将审计追踪记录写入某个日志表
- 进行额外的验证并根据需要回退数据
- 计算计算列的值或更新时间戳


不同的数据库触发器语法差别很大,具体参考具体文档


### 数据库安全
大多数数据库都给管理员提供了管理机制, 利用管理机制授予或限制对数据的访问

一般来说,需要保护的数据操作有:
- 对于数据库管理功能的访问
- 对特定数据库或表的访问
- 访问的类型
- 仅通过视图或储存过程对表进行访问
- 创建多层次的安全措施, 从而允许许多机遇登陆的访问和控制
- 限制管理用户的能力

安全性使用 SQL  的 GRANT 和 REVOKE 语句来进行管理
