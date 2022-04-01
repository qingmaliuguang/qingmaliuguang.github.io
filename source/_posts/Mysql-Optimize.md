---
title: Mysql 优化
date: 2021-08-17 18:54:41
tags: Mysql
---

# 索引

## 窄索引与宽索引

- 窄索引


- 宽索引：包含查询中所需要的全部数据列。


![Thin-Index-and-Fat-Index](/Users/luoxue/Desktop/images/mysql-Thin-Index-and-Fat-Index.jpg)

​	对于查询 `SELECT id, username, age FROM users WHERE username="draven"` 来说，(id, username) 就是一个窄索引，因为该索引没有包含存在于 SQL 查询中的 age 列，而 (id, username, age) 就是该查询的一个宽索引了，它**包含这个查询中所需要的全部数据列**。

​	宽索引能够避免二次的随机 IO，而窄索引就需要在对索引进行顺序读取之后再根据主键 id 从主键索引中查找对应的数据：

![Thin-Index-and-Clustered-Index](/Users/luoxue/Desktop/images/mysql-Thin-Index-and-Clustered-Index.jpg)

​	对于窄索引，每一个在索引中匹配到的记录行最终都需要执行另外的随机读取从聚集索引中获得剩余的数据，如果结果集非常大，那么就会导致随机读取的次数过多进而影响性能。

### 过滤因子

索引片变薄。

一个 SQL 查询扫描的索引片大小其实是由过滤因子决定的，也就是满足查询条件的记录行数所占的比例：

![Filter-Facto](/Users/luoxue/Desktop/images/mysql-Filter-Factor.jpg)

对于 users 表来说，sex=“male” 就不是一个好的过滤因子，它会选择整张表中一半的数据，所以**在一般情况下**我们最好不要使用 sex 列作为整个索引的第一列；而 name=“draven” 的使用就可以得到一个比较好的过滤因子了，它的使用能过滤整个数据表中 99.9% 的数据；当然我们也可以将这三个过滤进行组合，创建一个新的索引 (name, age, sex) 并同时使用这三列作为过滤条件：

![Combined-Filter-Facto](/Users/luoxue/Desktop/images/mysql-Combined-Filter-Factor.jpg)

> 当三个过滤条件都是**等值谓词**时，几个索引列的顺序其实是无所谓的，**索引列的顺序不会影响同一个 SQL 语句对索引的选择***，也就是索引 (name, age, sex) 和 (age, sex, name) 对于上图中的条件来说是完全一样的，这两个索引在执行查询时都有着完全相同的效果。

这种直接使用乘积来计算组合条件的过滤因子其实有一个比较重要的问题：列与列之间不应该有太强的相关性。

对于一张表中的同一个列，不同的值也会有不同的过滤因子，这也就造成了同一列的不同值最终的查询性能也会有很大差别：

![Same-Columns-Filter-Facto](/Users/luoxue/Desktop/images/mysql-Same-Columns-Filter-Factor.jpg)

当我们评估一个索引是否合适时，需要考虑极端情况下查询语句的性能，比如 0% 或者 50% 等；最差的输入往往意味着最差的性能，在平均情况下表现良好的 SQL 语句在极端的输入下可能就完全无法正常工作，这也是在设计索引时需要注意的问题。

总而言之，需要扫描的索引片的大小对查询性能的影响至关重要，而扫描的索引记录的数量，就是总行数与组合条件的过滤因子的乘积，索引片的大小最终也决定了从表中读取数据所需要的时间。

### 匹配列与过滤列

假设在 users 表中有 name、age 和 (name, sex, age) 三个辅助索引；当 WHERE 条件中存在类似 age = 21 或者 name = “draven” 这种**等值谓词**时，它们都会成为**匹配列**（Matching Column）用于选择索引树中的数据行，但是当我们使用以下查询时：

```sql
SELECT * FROM users
WHERE name = "draven" AND sex = "male" AND age > 20;
```

虽然我们有 (name, sex, age) 索引包含了上述查询条件中的全部列，但是在这里只有 name 和 sex 两列才是匹配列，MySQL 在执行上述查询时，会选择 name 和 sex 作为匹配列，扫描所有满足条件的数据行，然后将 age 当做**过滤列**（Filtering Column）：

![Match-Columns-Filter-Columns](/Users/luoxue/Desktop/images/mysql-Match-Columns-Filter-Columns.jpg)

**过滤列虽然不能够减少索引片的大小，但是能够减少从表中随机读取数据的次数**，所以在索引中也扮演着非常重要的角色。

## 索引的设计

从总体来看如何**减少随机读取的次数**是设计索引时需要重视的最重要的问题。

### 三星索引

三星索引是对于一个查询语句可能的最好索引，如果一个查询语句的索引是三星索引，那么它只需要进行**一次磁盘的随机读及一个窄索引片的顺序扫描**就可以得到全部的结果集。

![Three-Star-Index](/Users/luoxue/Desktop/images/mysql-Three-Star-Index.jpg)

为了满足三星索引中的三颗星，我们分别需要做以下几件事情：

1. 第一颗星需要取出**所有等值谓词中的列**，作为索引开头的最开始的列（任意顺序）；
2. 第二颗星需要将 **ORDER BY 列**加入索引中；
3. 第三颗星需要将**查询语句剩余的列**全部加入到索引中。

三星索引中每颗星都有其意义：

![Behind-Three-Star-Index](/Users/luoxue/Desktop/images/mysql-Behind-Three-Star-Index.jpg)

1. 第一颗星不只是将等值谓词的列加入索引，它的作用是减少索引片的大小以减少需要扫描的数据行；
2. 第二颗星用于避免排序，减少磁盘 IO 和内存的使用；
3. 第三颗星用于避免每一个索引对应的数据行都需要进行一次随机 IO 从聚集索引中读取剩余的数据。

在实际场景中，问题往往没有这么简单，我们虽然可以总能够通过宽索引避免大量的随机访问，但是在一些复杂的查询中（如同时拥有范围谓词和order by时）我们无法同时获得第一颗星和第二颗星。

总而言之，在设计单表的索引时，首先把查询中所有的**等值谓词全部取出**以任意顺序放在索引最前面，在这时，如果索引中同时存在范围索引和 ORDER BY 就需要权衡利弊了，希望最小化扫描的索引片厚度时，应该将**过滤因子最小的范围索引列**加入索引，如果希望避免排序就选择 **ORDER BY 中的全部列**，在这之后就只需要将查询中**剩余的全部列**加入索引了，通过这种固定的方法和逻辑就可以最快地获得一个查询语句的二星或者三星索引了。

### 访问

当 MySQL 读取一个索引行或者一个表行时，就会发生一次访问，当使用全表扫描或者扫描索引片时，**读取的第一个行就是随机访问**，随机访问需要磁盘进行寻道和旋转，所以其代价巨大，而接下来顺序读取的所有行都是通过顺序访问读取的，代价只有随机访问的千分之一。



# Mysql 如何使用索引

大多数 MySQL 索引（`PRIMARY KEY`、 `UNIQUE`、`INDEX`和 `FULLTEXT`）都存储在 [B 树中](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_b_tree)。例外：空间数据类型的索引使用 R 树；`MEMORY` 表也支持[哈希索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_hash_index)；`InnoDB`对`FULLTEXT`索引使用倒排列表。

通常，索引的使用如下面的讨论中所述。[第 8.3.9 节“B 树和哈希索引的比较”](https://dev.mysql.com/doc/refman/8.0/en/index-btree-hash.html)`MEMORY`中描述了特定于哈希索引（如表中使用的 ）的特征 。

MySQL 对这些操作使用索引：

- WHERE快速查找与**子句匹配**的行。

- 从考虑中排除行。如果有多个索引之间的选择，MySQL通常使用找到**最少行数**的索引（最具 [选择性的](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_selectivity)索引）。

- 如果表具有多列索引，则优化器可以使用索引的任何**最左边的前缀**来查找行。举例来说，如果你有一个三列的索引 `(col1, col2, col3)`，你有索引的搜索功能`(col1)`， `(col1, col2)`以及`(col1, col2, col3)`。有关更多信息，请参阅 [第 8.3.6 节，“多列索引”](https://dev.mysql.com/doc/refman/8.0/en/multiple-column-indexes.html)。

- 在执行连接时从其他表中检索行。如果**将列声明为相同的类型和大小**，MySQL 可以更有效地使用列上的索引。在这种情况下， [`VARCHAR`](https://dev.mysql.com/doc/refman/8.0/en/char.html)与 [`CHAR`](https://dev.mysql.com/doc/refman/8.0/en/char.html)被认为是相同的，如果它们被声明为相同的大小。例如， `VARCHAR(10)`和 `CHAR(10)`是相同的大小，但 `VARCHAR(10)`和 `CHAR(15)`不是。

  对于非二进制字符串列之间的比较，两列应使用相同的字符集。例如，将一`utf8`列与一 `latin1`列进行比较排除了索引的使用。

  如果不进行转换就无法直接比较值，则不同列的比较（例如，将字符串列与临时或数字列进行比较）可能会阻止使用索引。对于给定的值，如`1` 在数值列，它可能比较等于在字符串列，例如任何数量的值 `'1'`，`' 1'`， `'00001'`，或`'01.e1'`。这排除了对字符串列使用任何索引的可能性。

- 查找特定索引列的[`MIN()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_min)或 [`MAX()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_max)值*`key_col`*。这是由预处理器优化的，该预处理器检查您是否 在索引中之前出现的所有关键部分上使用。在这种情况下，MySQL 对每个or 表达式执行单个键查找，并将其替换为常量。如果所有表达式都替换为常量，则查询立即返回。例如： `WHERE *`key_part_N`* = *`constant`*`*`key_col`*[`MIN()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_min)[`MAX()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_max)

  ```sql
  SELECT MIN(key_part2),MAX(key_part2)
    FROM tbl_name WHERE key_part1=10;
  ```

- 如果排序或分组是在可用索引的最左前缀（例如，）上完成的，则对表进行排序或分组 。如果所有关键部分后跟，则以相反的顺序读取密钥。（或者，如果索引是降序索引，[则按](https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html)前向顺序读取键。）请参阅 [第 8.2.1.16 节“ORDER BY 优化”](https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html)、 [第 8.2.1.17 节“GROUP BY 优化”](https://dev.mysql.com/doc/refman/8.0/en/group-by-optimization.html)和 [第 8.3.13 节“降序索引”](https://dev.mysql.com/doc/refman/8.0/en/descending-indexes.html)。 `ORDER BY *`key_part1`*, *`key_part2`*``DESC`

- 在某些情况下，可以优化查询以在不咨询数据行的情况下检索值。（为查询提供所有必要结果的[索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_covering_index)称为 [覆盖索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_covering_index)。）如果查询仅使用某个索引中包含的表中的列，则可以从索引树中检索所选值以提高速度：

  ```sql
  SELECT key_part3 FROM tbl_name
    WHERE key_part1=1
  ```

**索引对于小表或大表的查询不太重要**，其中报告查询处理大部分或所有行。**当查询需要访问大部分行时，顺序读取比通过索引读取要快。顺序读取最小化磁盘搜索，即使查询不需要所有行。**有关详细信息[，](https://dev.mysql.com/doc/refman/8.0/en/table-scan-avoidance.html)请参阅[第 8.2.1.23 节，“避免全表扫描”](https://dev.mysql.com/doc/refman/8.0/en/table-scan-avoidance.html)。

# Explain

Explain输出列

| Column        | JSON Name     | Meaning                                                      |
| :------------ | :------------ | :----------------------------------------------------------- |
| id            | select_id     | 该SELECT语句的标识符                                         |
| select_type   | None          | 该SELECT语句的类型                                           |
| table         | table_name    | 表名                                                         |
| partitions    | partitions    | 查询将从其中匹配记录的分区。对于非分区表，该值为NULL。       |
| type          | access_type   | join 类型                                                    |
| possible_keys | possible_keys | 指示MySQL可以选择从哪些索引中查找该表中的行。                |
| key           | key           | 表示MySQL实际决定使用的键(索引)。如果MySQL决定使用一个possible_keys索引来查找行，该索引将作为键值列出。 |
| key_len       | key_length    | key_len列表示MySQL决定使用的键的长度。key_len的值使您能够确定MySQL实际使用的多部分键的多少部分。如果key列为NULL, key_len列也为NULL。 |
| ref           | ref           | ref列显示哪些列或常量与key列中指定的索引进行比较，以从表中选择行。<br/>如果值是func，则使用的值是某个函数的结果。<br/>要查看哪个函数，请使用EXPLAIN后面的SHOW WARNINGS查看扩展的EXPLAIN输出。这个函数实际上可能是一个运算符，比如算术运算符。 |
| rows          | rows          | 示MySQL认为执行查询时必须检查的行数。<br/>对于InnoDB表，这个数字是一个估计，可能并不总是准确的。 |
| filtered      | filtered      | 按表条件过滤的行百分比。                                     |
| Extra         | None          | 这一列包含关于MySQL如何解析查询的附加信息。                  |

select_type的值

| select_type Value    | JSON Name                  | Meaning                                                      |
| :------------------- | :------------------------- | :----------------------------------------------------------- |
| SIMPLE               | None                       | 简单的 SELECT (没有使用 UNION 或者 子查询)                   |
| PRIMARY              | None                       | SELECT中若包含任何复杂的子部分，最外层SELECT则被标记为Primary |
| UNION                | None                       | UNION语句中的第二个或之后的 SELECT 语句                      |
| DEPENDENT UNION      | dependent (true)           | 一个 UNION 语句中的第二个或之后的SELECT语句, 依赖于外层查询。 |
| UNION RESULT         | union_result               | UNION 的结果                                                 |
| SUBQUERY             | None                       | 子查询中的第一个 SELECT。                                    |
| DEPENDENT SUBQUERY   | dependent (true)           | 子查询中的第一个 SELECT, 依赖于外层查询。                    |
| DERIVED              | None                       | 在FROM列表中包含的子查询被标记为DERIVED(衍生);MySQL会递归执行这些子查询, 把结果放在临时表里。 |
| DEPENDENT DERIVED    | dependent (true)           | 依赖于另一张表的衍生表。                                     |
| MATERIALIZED         | materialized_from_subquery | 物化子查询                                                   |
| UNCACHEABLE SUBQUERY | cacheable (false)          | 无法缓存其结果的子查询，必须对外部查询的每一行重新求值。     |
| UNCACHEABLE UNION    | cacheable (false)          | 属于非缓存子查询的UNION中的第二或之后的SELECT。              |

> 通过物化来优化子查询
>
> 优化器使用物化能够更有效的来处理子查询。物化通过将子查询结果作为一个临时表来加快查询执行速度，正常来说是在内存中的。mysql第一次需要子查询结果是，它物化结果到一张**临时表**中。在之后的任何地方需要该结果集，mysql会再次引用临时表。优化器也许会使用一个哈希索引来使得查询更快速代价更小。索引是唯一的，排除重复并使得表数据更少。
> 子查询物化如果可以的话会使用一个内存临时表，如果表太大则会落实为磁盘存储。具体请看8.4.4的在mysql中使用内部临时表。

**EXPLAIN 输出的 type 列描述表是如何连接的。**在json格式的输出中，可以找到access_type属性的值。下面的列表描述了连接类型，按**从好到坏**的顺序排列:

- system

  - 该表只有一行(= system table)。这是const连接类型的特殊情况。

- const

  - 该表最多有一个匹配的行，在查询的开始读取。因为只有一行，所以这一行中的列的值可以被优化器的其余部分视为常量。const表非常快，因为它们只被读取一次。

  - 当将**PRIMARY KEY**或**UNIQUE索引**的所有部分与**常量**进行比较时，使用const。在下面的查询中，tbl_name可以用作const表:

    ```sql
    SELECT * FROM tbl_name WHERE primary_key=1;
    
    SELECT * FROM tbl_name
      WHERE primary_key_part1=1 AND primary_key_part2=2;
    ```

- eq_ref

  - 对于前面表中的每个行组合，将从该表中读取一行。除了系统类型和const类型，这是最好的连接类型。当连接使用索引的所有部分，并且**索引是主键或UNIQUE NOT NULL索引**时使用。

  - eq_ref可以用于使用=操作符进行比较的索引列。比较值可以是常量或表达式，该表达式使用在该表之前读取的表中的列。在下面的例子中，MySQL可以使用eq_ref连接来处理ref_table:

    ```sql
    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column=other_table.column;
    
    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column_part1=other_table.column
      AND ref_table.key_column_part2=1;
    ```

- ref

  - 对于前面表中的每个行组合，将从该表中读取具有匹配索引值的所有行。如果连接仅使用键的最左边前缀，或者该键不是PRIMARY key或UNIQUE索引(换句话说，**如果连接不能根据键值选择单行**)，则使用ref。如果所使用的键只匹配少数行，那么这是一种很好的连接类型。

  - ref可用于使用=或<=>操作符进行比较的索引列。在下面的例子中，MySQL可以使用ref连接来处理ref_table:

    ```sql
    SELECT * FROM ref_table WHERE key_column=expr;
    
    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column=other_table.column;
    
    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column_part1=other_table.column
      AND ref_table.key_column_part2=1;
    ```

- fulltext

  - 使用FULLTEXT索引执行的join。

- ref_or_null

  - 这个连接类型类似于ref，但是MySQL会额外搜索包含NULL值的行。这种连接类型优化最常用于解析子查询。在下面的例子中，MySQL可以使用ref_or_null连接来处理ref_table:

    ```sql
    SELECT * FROM ref_table
      WHERE key_column=expr OR key_column IS NULL;
    ```

- index_merge

  - 此连接类型表明使用了Index Merge优化。在本例中，输出行中的key列包含所使用索引的列表，key_len包含所使用索引的最长key部分的列表。

- unique_subquery

  - 这个类型替换了以下形式的IN子查询的eq_ref:

    ```sql
    value IN (SELECT primary_key FROM single_table WHERE some_expr)
    ```

  - unique_subquery只是一个**索引查找函数**，它完全替代子查询以提高效率。

- index_subquery

  - 这种连接类型类似于unique_subquery。它代替了IN子查询，但它适用于以下形式的子查询中的**非唯一索引**:

    ```sql
    value IN (SELECT key_column FROM single_table WHERE some_expr)
    ```

- range

  - 只检索给定范围内的行，使用索引选择行。输出行中的key列指示使用哪个索引。key_len包含所使用的最长的key部分。对于这种类型，ref列是NULL。

  - 当使用任意的=，<>，>，>=，<，<=，is NULL， <=>， BETWEEN, LIKE, or IN()操作符将key列与常量进行比较时，可以使用range:

    ```sql
    SELECT * FROM tbl_name
      WHERE key_column = 10;
    
    SELECT * FROM tbl_name
      WHERE key_column BETWEEN 10 and 20;
    
    SELECT * FROM tbl_name
      WHERE key_column IN (10,20,30);
    
    SELECT * FROM tbl_name
      WHERE key_part1 = 10 AND key_part2 IN (10,20,30);
    ```

- index

  - index 连接类型与ALL相同，只是扫描的是索引树。这有两种方式:

    - 如果索引是查询的覆盖索引，并且可以用来满足表中所需的所有数据，则只扫描索引树。在本例中，Extra 列显示"*Using index*"。仅索引扫描通常比ALL扫描快，因为索引的大小通常小于表数据。
    - 全表扫描是通过从索引中读取数据以按照索引顺序查找数据行来执行的。"*Using index*"不出现在 Extra 列。

    当查询只使用属于单个索引的列时，MySQL可以使用这种连接类型。

- all

  - 对前面表中的每个行组合进行全表扫描。如果表是第一个没有标记为const的表，这通常不太好，在所有其他情况下通常都很糟糕。通常，您可以通过添加索引来避免ALL，这些索引支持基于常量值或早期表中的列值从表中检索行。



# mysqldumpslow 慢查询日志

MySQL 慢查询日志包含有关需要很长时间执行的查询的信息。mysqldumpslow解析 MySQL 慢查询日志文件并汇总其内容。

```sql
mysqldumpslow [*options*] [*log_file* ...]
```

mysqldumpslow 支持以下 options：

| Option Name | Description                                           |
| :---------- | :---------------------------------------------------- |
| -a          | 不要把所有数字抽象为N，字符串抽象为S。                |
| -n          | 名称中至少包含N位的抽象数字。                         |
| --debug     | 以debug模式运行，输出debug信息。                      |
| -g          | 只考虑与模式匹配的语句。                              |
| --help      | 打印help信息并退出。                                  |
| -h          | 日志文件名中添加服务器的主机名。                      |
| -i          | server 实例的名称。                                   |
| -l          | 不从总时间中减去 lock 的时间。                        |
| -r          | 反向排序                                              |
| -s          | 如何对输出进行排序，sort_type：t、at、l、al、r、ar、c |
| -t          | 在输出中只显示前N个查询。                             |
| --verbose   | 详细模式。                                            |



# semi-join

semi-join Materialization 是用于semi-join的一种特殊的子查询物化技术。通常包含两种策略：

1.Materialization/lookup

2.Materialization/scan

考虑一个查询欧洲有大城市的国家:

```sql
select * from Country
where Country.code IN (select City.Country
                       from City
                       where City.Population > 7*1000*1000)
      and Country.continent='Europe'
```

子查询是非相关子查询。也即是我们可以独立运行内查询。semi-materialization的思想是使用city.country中可能的值填充一个临时表，然后和欧洲的国家进行关联。

![img](/Users/luoxue/Desktop/images/mysql-example for semi-join.png)

这个join可以从两个方向进行：

1. 从物化表到国家表
2. 从国家表到物化表

第一个方向涉及一个全表扫描(在物化表上的全表扫描)，因此被称为"Materialization-scan"；如果从第二个方向进行，最廉价的方式是使用主键从物化表中lookup出匹配的记录。这种方式被称为"Materialization-lookup"。



# 分区

本节讨论MySQL 8.0中可用的分区类型。这些包括这里列出的类型:

- RANGE partitioning
  - 这种类型的分区根据位于给定范围内的列值将行分配给分区。
- LIST partitioning
  - 类似于RANGE分区，但分区是根据匹配一组离散值之一的列。
- HASH partitioning
  - 使用这种类型的分区，将根据用户定义表达式返回的值选择分区，该表达式对要插入到表中的行中的列值进行操作。这个函数可以包含任何在MySQL中有效的表达式，并产生一个非负整数值。该类型的扩展 LINEAR HASH 也可用。
- KEY partitioning
  - 这种类型的分区类似于HASH分区，除了只提供一个或多个要计算的列，而且MySQL服务器提供自己的哈希函数。这些列可以包含非整数值，因为MySQL提供的散列函数保证了一个整数结果，而不管列数据类型是什么。这种类型的扩展，LINEAR KEY，也可用。

数据库分区的一个非常常见的用途是按日期隔离数据。一些数据库系统支持显式的日期分区，MySQL在8.0中没有实现。然而，在MySQL中创建基于DATE、TIME或DATETIME列或基于使用这些列的表达式的分区方案并不困难。

当使用KEY或LINEAR KEY进行分区时，可以使用DATE、TIME或DATETIME列作为分区列，而不需要对列值进行任何修改。例如，这个表创建语句在MySQL中是完全有效的:

```sql
CREATE TABLE members (
    firstname VARCHAR(25) NOT NULL,
    lastname VARCHAR(25) NOT NULL,
    username VARCHAR(16) NOT NULL,
    email VARCHAR(35),
    joined DATE NOT NULL
)
PARTITION BY KEY(joined)
PARTITIONS 6;
```

在MySQL 8.0中，也可以使用DATE或DATETIME列作为使用RANGE列和LIST列分区的分区列。

其他分区类型需要生成整数值或NULL的分区表达式。如果你想使用RANGE、LIST、HASH或LINEAR HASH的基于日期的分区，你可以简单地使用一个函数，在DATE、TIME或DATETIME列上操作并返回这样一个值，如下所示:

```sql
CREATE TABLE members (
    firstname VARCHAR(25) NOT NULL,
    lastname VARCHAR(25) NOT NULL,
    username VARCHAR(16) NOT NULL,
    email VARCHAR(35),
    joined DATE NOT NULL
)
PARTITION BY RANGE( YEAR(joined) ) (
    PARTITION p0 VALUES LESS THAN (1960),
    PARTITION p1 VALUES LESS THAN (1970),
    PARTITION p2 VALUES LESS THAN (1980),
    PARTITION p3 VALUES LESS THAN (1990),
    PARTITION p4 VALUES LESS THAN MAXVALUE
);
```

使用日期进行分区的其他例子可在本章的以下章节中找到:

- [Section 24.2.1, “RANGE Partitioning”](https://dev.mysql.com/doc/refman/8.0/en/partitioning-range.html)
- [Section 24.2.4, “HASH Partitioning”](https://dev.mysql.com/doc/refman/8.0/en/partitioning-hash.html)
- [Section 24.2.4.1, “LINEAR HASH Partitioning”](https://dev.mysql.com/doc/refman/8.0/en/partitioning-linear-hash.html)

有关更复杂的基于日期的分区示例，请参见以下部分:

- [Section 24.4, “Partition Pruning”](https://dev.mysql.com/doc/refman/8.0/en/partitioning-pruning.html)
- [Section 24.2.6, “Subpartitioning”](https://dev.mysql.com/doc/refman/8.0/en/partitioning-subpartitions.html)

MySQL分区通过TO_DAYS()、YEAR()和TO_SECONDS()函数进行了优化。但是，您可以使用其他返回整数或NULL的日期和时间函数，如WEEKDAY()、DAYOFYEAR()或MONTH()。

重要的是要记住—不管您使用的分区类型是什么—分区总是在创建时自动编号并按顺序编号，从0开始。当将新行插入分区表时，将使用这些分区号来标识正确的分区。例如，如果您的表使用4个分区，则这些分区编号为0、1、2和3。对于RANGE和LIST分区类型，必须确保为每个分区号定义了一个分区。对于HASH分区，用户提供的表达式必须计算为大于0的整数值。对于KEY分区，这个问题是由MySQL服务器内部使用的哈希函数自动处理的。

分区的名称通常遵循管理其他MySQL标识符(如表和数据库标识符)的规则。但是，您应该注意分区名称不区分大小写。例如，下面的CREATE TABLE语句失败，如下所示:

```sql
mysql> CREATE TABLE t2 (val INT)
    -> PARTITION BY LIST(val)(
    ->     PARTITION mypart VALUES IN (1,3,5),
    ->     PARTITION MyPart VALUES IN (2,4,6)
    -> );
ERROR 1488 (HY000): Duplicate partition name mypart
```

因为MySQL看不到分区名mypart和mypart之间的区别，所以会发生失败。

当您指定表的分区数时，必须将其表示为一个正的、不带前导零的非零整数字面值，并且可能不是像0.8E+01或6-2这样的表达式，即使它计算出的值是一个整数值。小数是不允许的。





------

> 感谢摘抄（或参考）的文章：
>
> [MySQL 索引设计概要](https://draveness.me/sql-index-intro/)
>
> [8.3.1 How MySQL Uses Indexes](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html)
>
> [8.8.2 EXPLAIN Output Format](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-join-types)
>
> [semi-join子查询优化 -- semi-join Materialization策略](https://www.cnblogs.com/abclife/p/10899350.html)
>
> [4.6.10 mysqldumpslow — Summarize Slow Query Log Files](https://dev.mysql.com/doc/refman/8.0/en/mysqldumpslow.html)
>
> [24.2 Partitioning Types](https://dev.mysql.com/doc/refman/8.0/en/partitioning-types.html)

