> 大部分文档翻译自官方文档，部分来源于网络，如有侵权，请及时告知，谢谢！

### 1、什么是索引

数据库索引是一种提高表中操作速度的数据结构。 可以使用一列或多列创建索引，为快速随机查找和对记录的访问的有效排序提供基础。

实际上，索引也是一种表，它将主键或索引字段以及指向每个记录的指针保存到实际表中。

INSERT和UPDATE语句在具有索引的表上花费更多时间，而SELECT语句在这些表上变得快速。 原因是在进行插入或更新时，数据库也需要插入或更新索引值。


### 2、MySQL如何使用索引

> 参考资料：(How MySQL Uses Indexes)[https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html]

索引是用于快速查找指定列值的行。没有索引的话，MySQL必须从第一行开始然后读取整个表去查找相关行。

大部分索引存储在B-trees；除了空间类型数据索引使用R-trees；MEMORY表也支持哈希索引; InnoDB使用FULLTEXT索引的反转列表。

以下这些操作MySQL使用索引：

* 01、快速查找匹配`where`语句的行；
* 02、消除行的考虑。 如果在多个索引之间有选择，MySQL通常使用找到最小行数（最具选择性的索引）的索引。
* 03、如果表具有多列索引（联合索引），则优化程序可以使用索引的任何最左前缀来查找行。 例如，如果在（col1，col2，col3）上有三列索引，则在（col1），（col1，col2）和（col1，col2，col3）上编制索引搜索功能。
* 04、在执行连接时从其他表中检索行。 如果声明它们的类型和大小相同，MySQL可以更有效地使用列上的索引。 在此上下文中，如果将VARCHAR和CHAR声明为相同大小，则认为它们是相同的。 例如，VARCHAR（10）和CHAR（10）的大小相同，但VARCHAR（10）和CHAR（15）不是。
    * 0401、对于非二进制字符串列之间的比较，两列应使用相同的字符集。 例如，将utf8列与latin1列进行比较会排除使用索引。
    * 0402、不相似列的比较（例如，将字符串列与时间或数字列进行比较）可能会在没有转换的情况下无法直接比较值时阻止使用索引。 对于给定值（例如数字列中的1），它可能比较等于字符串列中的任何数量的值，例如“1”，“1”，“00001”或“01 .e1”。 这排除了对字符串列的任何索引的使用。
* 05、查找特定索引列`key_col`的`MIN（）`或`MAX（）`值。 这是由预处理器优化的，该预处理器检查您是否在索引中的`key_col`之前出现的所有关键部分上使用`WHERE key_part_N =常量`。 在这种情况下，MySQL对每个MIN（）或MAX（）表达式执行单个键查找，并用常量替换它。 如果所有表达式都替换为常量，则查询立即返回。
* 06、如果对可用索引的最左前缀（例如，ORDER BY key_part1，key_part2）进行排序或分组，则对表进行排序或分组。 如果所有关键部分后面都是DESC，则按相反顺序读取密钥。 （或者，如果索引是降序索引，则按正向顺序读取密钥。）请参见第8.2.1.15节“ORDER BY优化”，第8.2.1.16节“GROUP BY优化”和第8.3.13节“ 降序指数“。
* 07、在某些情况下，可以优化查询以在不咨询数据行的情况下检索值。 （为查询提供所有必要结果的索引称为覆盖索引。）如果查询仅使用表中包含某些索引的列，则可以从索引树中检索所选值以获得更快的速度。

对于小型表或报表查询处理大多数或所有行的大型表的查询，索引不太重要。当查询需要访问大多数行时，顺序读取比通过索引更快。顺序读取可以最大限度地减少磁盘搜索，即使查询不需要所有行也是如此。

### 3、如何新建MySQL索引

（1）创建索引语法
```
CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX index_name
    [index_type]
    ON tbl_name (key_part,...)
    [index_option]
    [algorithm_option | lock_option] ...

key_part: {col_name [(length)] | (expr)} [ASC | DESC]

index_option:
    KEY_BLOCK_SIZE [=] value
  | index_type
  | WITH PARSER parser_name
  | COMMENT 'string'
  | {VISIBLE | INVISIBLE}

index_type:
    USING {BTREE | HASH}

algorithm_option:
    ALGORITHM [=] {DEFAULT | INPLACE | COPY}

lock_option:
    LOCK [=] {DEFAULT | NONE | SHARED | EXCLUSIVE}
```

* 通常，在使用`CREATE TABLE`创建表的时候会创建所有的索引。

* `CREATE INDEX`映射到`ALTER TABLE`语句以创建索引。`CREATE     INDEX`不能用来创建主键，`ALTER TABLE`可以。

* 联合索引格式为：`(key_part1, key_part2, ...)`创建多值索引，只要满足最左前缀原则，即可使用联合索引。

* `key_part`以ASC或DESC结尾指定是否索引按照升序或降序存储。

（2）不同类型索引

* Column Prefix Key Parts 对于字符串列，索引可以使用关键部分进行创建，使用`col_name(length)`语法指定索引前缀长度。
    * `CREATE INDEX part_of_name ON customer (name(10))`
* Unique Indexes（唯一索引），唯一索引创建一个约束，使索引中的值必须是唯一的。如果插入一个已经存在的值，则会产生错误。如果为唯一索引指定前缀值，则列值必须在前缀长度内是唯一的。唯一索引允许包含NULL的列有多个NULL值。
* Full-Text Indexes（全文索引），全文索引支持只包含CHAR、VARCHAR、TEXT列的InnoDB和MyISAM表。索引总是产生在整列上，列前缀索引不支持，而且任何被指定的列前缀都是被忽略的。
* Spatial Indexes（空间索引）：MyISAM，InnoDB，NDB和ARCHIVE存储引擎支持POINT和GEOMETRY等空间列。

（3）创建索引可选项

* KEY_BLOCK_SIZE [=] value
* index_type
* WITH PARSER parser_name
* COMMENT 'string'
* VISIBLE, INVISIBLE

（4）表复制和索引选项

可以给出ALGORITHM和LOCK子句来影响表复制方法和在修改索引时读写表的并发级别。

### 3、MySQL索引性能优化神器

（1）EXPALIN命令

`EXPLAIN`语句提供MySQL如何执行语句的信息。

* `EXPLAIN`在`SELECT`、`DELETE`、`INSERT`、`REPLACE`和`UPDATE`语句一起使用时生效。

（2）EXPLAIN输出结果解析

A、EXPLAIN输出结果

```
# tasks表
# 索引信息如下
# unique key id
# key idx_taskState(`task_state`), key idx_project(`project_id`), key idx_taskAlias(`task_alias`)

# sql语句如下

EXPLAIN SELECT
	task_id 
FROM
	airflow_service.tasks 
WHERE
	project_id = 3;
```

输出结果为：

```
id: 1
select_type: SIMPLE
table: tasks
partitions: NULL
type: ALL
possible_keys: idx_project
key: NULL
key_len: NULL
ref: NULL
rows: 233
filtered: 88.41
Extra: Using where
```

B、`EXPLAIN`输出结果的含义

> 参考资料：[官方文档](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)

列 | JSON名称 | 含义
---|---|---
id | select_id | 查询标识符
select_type | None | 查询类型
table | table_name | 查询使用的表
partitions | partitions | 匹配分区
type | access_type | join类型
possible_keys | possible_keys | 选择可能使用的索引
key | key | 实际被选择的索引
key_len | key_length | 选择的key的长度之和
ref | ref | 与索引进行比较的列
rows | rows | 估计要检查的行数
filtered | filtered | 被表条件过滤行数百分比
Extra | None | 额外信息

**JSON属性为空在输出JSON格式的EXPLAIN结果不展示。**

* id：查询标识符，这是查询中的`SELECT`串号。如果行引用其他行的并集结果集，这个值可能为NULL。在这种情况下，table列显示类似<unionM, N>的值，表示该行引用id值为M和N和的并集。
* type：join类型
    * system：该表只有一行（=系统表）。 这是const连接类型的特例。
    * const(常量、恒定的、不变的)
        * 该表最多匹配一行，在查询开头读取，所以优化器的其余部分可以将此行中列的值是为常量。const表非常快，因为它们只读一次。
        * 将PRIMARY_KEY或UNIQUE中的所有部分与常量值进行比较时使用const。
    * eq_ref
        * 对于join中的前一张表的每一行，从该表中匹配一行，除了system和const类型。
        * eq_ref可用于使用=运算符进行比较的索引列，比较直可以是常量，也开始是使用在此表之前读取的表中的列的表达式。
    * ref
        * 对于join中的前一张表的每个行组合，将从此表中读取具有匹配索引值的所有行。如果连接仅使用键的最左前缀或者键不是PRIMARY KEY或UNIQUE索引（换句话说，如果连接不能键值选择单行），则使用ref。
        * ref可用于=或<=>运算符进行比较的索引列。
    * fulltext：使用FULLTEXT索引执行连接。
    * ref_or_null：这种连接类型与ref类似，但附加的是MySQL对包含NULL值的行进行额外搜索。 此连接类型优化最常用于解析子查询。
    * index_merge：此连接类型表示使用了索引合并优化。 在这种情况下，输出行中的键列包含使用的索引列表，key_len包含所用索引的最长键列表。
    * unique_subquery：这种类型在以下格式的子查询中替换`eq_ref`。(`value IN (SELECT primary_key from single_table where some_expr)`)
    * index_subquery：这种join类型与`unique_subquery`类似。它取代了子查询，但是对于以下非唯一索引子查询生效。（`value IN (SELECT key_column FROM single_table WHERE some_expr)`）
    * range
        * 仅检索给定范围内的行，使用索引选择行。输出行中的`key`列指示使用哪个索引。输出行中的`key_len`列包含被使用的最长的key。这种类型`ref`列为NULL。
        * 使用=，<>，>，> =，<，<=，IS NULL，<=>，BETWEEN，LIKE或IN（）运算符中的任何一个将键列与常量进行比较时，可以使用`range`。
    * index：这种类型与`ALL`相同，除了索引树会被扫描。有两种方式：
        * 如果索引是查询的覆盖索引，并且可用于满足表中所需的所有数据，则仅扫描索引树。 在这种情况下，Extra列显示Using index。 仅索引扫描通常比ALL快，因为索引的大小通常小于表数据。
        * 使用索引中的读取执行全表扫描，以按索引顺序查找数据行。 Using index不会出现在Extra列中。
    * ALL：对前一张表中的每个行组合进行全表扫描。 如果表是第一个没有标记为const的表，这通常是不好的，并且在所有其他情况下通常非常糟糕。 通常，您可以通过添加索引来避免ALL，这些索引根据以前表中的常量值或列值从表中启用行检索。
* select_type：查询类型，可能为下述表格里展示中的任意一个值。

查询类型值 | JSON名称 | 含义
---|---|---
SIMPLE | None | 简单的SELECT（未使用UNION或子查询）
PRIMARY | None | 最外面的SELECT
UNION | None | UNION语句中的第二个或更靠后的SELECT
DEPENDENT UNION | dependent（true）| UNION语句中的第二个或更靠后的SELECT，依赖于外部SELECT
UNION RESULT | union_result| UNION的结果
SUBQUERY | None | 子查询中的第一个SELECT
DEPENDENT SUBQUERY | dependent（true）| 子查询中的第一个SELECT，依赖于外部SELECT
DERIVED | None | 临时表
MATERIALIZED | materialize_from_subquery | Marterialized subquery(物化子查询？？)
UNCACHEABLE SUBQUERY | cacheable(false) | 无法缓存结果的子查询，而且必须为外部查询每一行重新计算
UNCACHEABLE UNION | cacheable(false) | 在UNION中第二个或靠后的查询属于无法缓存的子查询（请参阅UNCACHEABLE SUBQUERY）

* table：输出行引用的表名称。这也可以为以下值之一。
    * <unionM,N>：引用id值为M和N的行的并集；
    * <derivedN>：引用的是id值为N的派生表的结果。例如：派生表可能来自`FROM`子句。
    * <subqueryN>: 引用id值为N的具体化子查询的结果。

* partitions：join类型。
* possible_keys：`possible_keys`列展示MySQL在这个表里查询行使用的索引。此列完全独立于`EXPLAIN`中输出表的顺序。这意味着`possible_keys`中的key在实际已生成的表顺序中无法被使用。
* key：MySQL实际使用的作为索引的key
* key_len：MySQL实际使用的作为索引的key的长度。`key_len`的值决定了MySQL实际使用了联合索引中的多少个。
* ref：ref列显示哪些列或常量与键列中指定的索引进行比较，以从表中选择行。
* rows
    * rows列表示MySQL认为必须检查以执行查询的行数。
    * 对于InnoDB表，此数字是估计值，可能并不总是准确的。
* filtered
    * filtered列表示按表条件进行过滤的行数的预估百分比。
    * 最大值为100，意味着不会对行进行过滤。
    * 值从100开始减少表示过滤比增加。rows显示检查的估计行数，rows * filterd表示将与后面的表连接的行数。例如，如果rows=100、filtered=50%，则与后面的表连接的行数为1000 * 50%=500。
* Extra：`Extra`列包含MySQL如何解析查询的额外信息。以下的列表解释了可能出现在这一列中的值。如果你想让你的查询尽可能地快，注意`Extra`列地值为Using filesort和Using temporary。
    * Using filesort：MySQL必须执行额外的传递以找出如何按排序顺序检索行。 排序是通过根据连接类型遍历所有行并将排序键和指针存储到与WHERE子句匹配的所有行的行来完成的。 然后对键进行排序，并按排序顺序检索行。
    * Using tempory：要解析查询，MySQL需要创建一个临时表来保存结果。 如果查询包含以不同方式列出列的GROUP BY和ORDER BY子句，则通常会发生这种情况。
    * Using where：WHERE子句用于限制哪些行与下一个表匹配或发送到客户端。 除非您特意打算从表中获取或检查所有行，否则如果Extra值不是Using where并且表连接类型为ALL或index，则查询可能会出错。
    * Using index：仅使用索引树中的信息从表中检索列信息，而不必执行额外的搜索以读取实际行。 当查询仅使用属于单个索引的列时，可以使用此策略。


#### 5、全表扫描而不使用索引的场景

* 01、如果表中的数据过少，可能会出现全表扫描而不使用索引，原因是：MySQL会估计全表扫描的成本可能低于首先匹配索引然后查找行；
* 02、如果join的两张表字符集(charset)不同会导致全表扫描；