

### postgres索引类型

PostgreSQL提供了多种索引类型： B-tree、Hash、GiST、SP-GiST 、GIN 和 BRIN。每一种索引类型使用了 一种不同的算法来适应不同类型的查询。默认情况下， `CREATE INDEX`命令创建适合于大部分情况的B-tree 索引。

B-tree可以在可排序数据上的处理等值和范围查询

Hash索引只能处理简单等值比较。不论何时当一个索引列涉及到一个使用了`=`操作符的比较时，查询规划器将考虑使用一个Hash索引。

```
CREATE INDEX name ON table USING HASH (column);
```

GiST索引也有能力优化“最近邻”搜索，例如：

```
SELECT * FROM places ORDER BY location <-> point '(101,456)' LIMIT 10;
```

和GiST相似，SP-GiST索引为支持多种搜索提供了一种基础结构。SP-GiST 允许实现众多不同的非平衡的基于磁盘的数据结构，例如四叉树、k-d树和radix树。

与 GiST 和 SP-GiST相似， GIN 可以支持多种不同的用户定义的索引策略，并且可以与一个 GIN 索引配合使用的特定操作符取决于索引策略。

BRIN 索引（块范围索引的缩写）存储有关存放在一个表的连续物理块范围上的值摘要信息。与 GiST、SP-GiST 和 GIN 相似，BRIN 可以支持很多种不同的索引策略，并且可以与一个 BRIN 索引配合使用的特定操作符取决于索引策略。

[各种索引介绍](https://greenplum.cn/2019/04/15/postgres-index/)

### 单列索引

假设我们有一个如下的表：

```
CREATE TABLE test1 (
    id integer,
    content varchar
);
```

而应用发出很多以下形式的查询：

```
SELECT content FROM test1 WHERE id = constant;
```

```
CREATE INDEX test1_id_index ON test1 (id);
```

### 多列索引

```
CREATE TABLE test2 (
  major int,
  minor int,
  name varchar
);
```

（即将我们的`/dev`目录保存在数据库中）而且我们经常会做如下形式的查询：

```
SELECT name FROM test2 WHERE major = constant AND minor = constant;
```

那么我们可以在`major`和`minor`上定义一个索引：

```
CREATE INDEX test2_mm_idx ON test2 (major, minor);
```

当然，要使索引起作用，查询条件中的列必须要使用适合于索引类型的操作符，使用其他操作符的子句将不会被考虑使用索引。

多列索引应该较少地使用。在绝大多数情况下，单列索引就足够了且能节约时间和空间。具有超过三个列的索引不太有用，除非该表的使用是极端程式化的

多列复合索引的创建建议：

1、离散查询条件（例如 等值）的列放在最前面，如果一个复合查询中有多个等值查询的列，尽量将选择性好（count(distinct) 值多的）的放在前面。

2、离散查询条件（例如 多值）的列放在后面，如果一个复合查询中有多个多值查询的列，尽量将选择性好（count(distinct) 值多的）的放在前面。

3、连续查询条件（例如 范围查询）的列放在最后面，如果一个复合查询中有多个多值查询的列，尽量将输入范围条件返回结果集少的列放前面，提高筛选效率（同时也减少索引扫描的范围）。

4、如果返回的结果集非常大（或者说条件命中率很高），并且属于流式返回（或需要高效率优先返回前面几条记录），同时有排序输出的需求。建议按排序键建立索引。

[多列索引查询优化建议](https://yq.aliyun.com/articles/582852)

### 索引和order by

规划器会考虑以两种方式来满足一个`ORDER BY`说明：扫描一个符合说明的可用索引，或者先以物理顺序扫描表然后再显式排序。对于一个需要扫描表的大部分的查询，一个显式的排序很可能比使用一个索引更快，因为其顺序访问模式使得它所需要的磁盘I/O更少。只有在少数行需要被取出时，索引才会更有用。一种重要的特殊情况是`ORDER BY`与`LIMIT` *n*联合使用：一个显式的排序将会处理所有的数据来确定最前面的*n*行，但如果有一个符合`ORDER BY`的索引，前*n*行将会被直接获取且根本不需要扫描剩下的数据。

默认情况下，B-tree索引将它的项以升序方式存储，并将空值放在最后。这意味着对列`x`上索引的一次前向扫描将产生满足`ORDER BY x`（或者更长的形式：`ORDER BY x ASC NULLS LAST`）的结果。索引也可以被后向扫描，产生满足`ORDER BY x DESC`（`ORDER BY x DESC NULLS FIRST`， `NULLS FIRST`是`ORDER BY DESC`的默认情况）。

我们可以在创建B-tree索引时通过`ASC`、`DESC`、`NULLS FIRST`和`NULLS LAST`选项来改变索引的排序，例如：

```
CREATE INDEX test2_info_nulls_low ON test2 (info NULLS FIRST);
CREATE INDEX test3_desc_index ON test3 (id DESC NULLS LAST);
```

一个以升序存储且将空值前置的索引可以根据扫描方向来支持`ORDER BY x ASC NULLS FIRST`或 `ORDER BY x DESC NULLS LAST`。

> order by 索引对于limit n操作有较大影响

显然，具有非默认排序的索引是相当专门的特性，但是有时它们会为特定查询提供巨大的速度提升。是否值得维护这样一个索引取决于我们会多频繁地使用需要特殊排序的查询。

> 频繁使用排序查询应该使用顺序索引

### 组合多个索引

只有查询子句中在索引列上使用了索引操作符类中的操作符并且通过`AND`连接时才能使用单一索引。例如，给定一个`(a, b)` 上的索引，查询条件`WHERE a = 5 AND b = 6`可以使用该索引，而查询`WHERE a = 5 OR b = 6`不能直接使用该索引。

幸运的是，PostgreSQL具有组合多个索引（包括多次使用同一个索引）的能力来处理那些不 能用单个索引扫描实现的情况。系统能在多个索引扫描之间安排`AND`和`OR`条件。例如， `WHERE x = 42 OR x = 47 OR x = 53 OR x = 99`这样一个查询可以被分解成为四个独立的在`x`上索引扫描，每一个扫描使用其中一个条件。这些查询的结果将被“或”起来形成最后的结果。另一个例子是如果我们在`x`和`y`上都有独立的索引，`WHERE x = 5 AND y = 6`这样的查询的一种可能的实现方式就是分别使用两个索引配合相应的条件，然后将结果“与”起来得到最后的结果行。

为了组合多个索引，系统扫描每一个所需的索引并在内存中准备一个*位图*用于指示表中符合索引条件的行的位置。然后这些位图会被根据查询的需要“与”和“或”起来。最后，实际的表行将被访问并返回。表行将被以物理顺序访问，因为位图就是以这种顺序布局的。这意味着原始索引中的任何排序都会被丢失，并且如果存在一个`ORDER BY`子句就需要一个单独的排序步骤。由于这个原因以及每一个附加的索引都需要额外的时间，即使有额外的索引可用，规划器有时也会选择使用单一索引扫描。

在所有的应用（除了最简单的应用）中，可能会有多种有用的索引组合，数据库开发人员必须做出权衡以决定提供哪些索引。有时候多列索引最好，但是有时更好的选择是创建单独的索引并依赖于索引组合特性。例如，如果我们的查询中有时只涉及到列`x`，有时候只涉及到列`y`，还有时候会同时涉及到两列，我们可以选择在x和y上创建两个独立索引然后依赖索引组合来处理同时涉及到两列的查询。我们当然也可以创建一个`(x, y)`上的多列索引。当查询同时涉及到两列时，该索引会比组合索引效率更高，但是正如[Section 11.3](http://www.postgres.cn/docs/11/indexes-multicolumn.html)中讨论的，它在只涉及到y的查询中几乎完全无用，因此它不能是唯一的一个索引。一个多列索引和一个`y`上的独立索引的组合将会工作得很好。多列索引可以用于那些只涉及到`x`的查询，尽管它比`x`上的独立索引更大且更慢。最后一种选择是创建所有三个索引，但是这种选择最适合表经常被执行所有三种查询但是很少被更新的情况。如果其中一种查询要明显少于其他类型的查询，我们可能需要只为常见类型的查询创建两个索引。

### 唯一索引

索引也可以被用来强制列值的唯一性，或者是多个列组合值的唯一性。

```
CREATE UNIQUE INDEX name ON table (column [, ...]);
```

当前，只有B-tree能够被声明为唯一。

当一个索引被声明为唯一时，索引中不允许多个表行具有相同的索引值。空值被视为不相同。一个多列唯一索引将会拒绝在所有索引列上具有相同组合值的表行。

PostgreSQL会自动为定义了一个唯一约束或主键的表创建一个唯一索引。该索引包含组成主键或唯一约束的所有列（可能是一个多列索引），它也是用于强制这些约束的机制。

#### Note

不需要手工在唯一列上创建索引，如果那样做也只是重复了自动创建的索引而已。

### 表达式索引

一个索引列并不一定是底层表的一个列，也可以是从表的一列或多列计算而来的一个函数或者标量表达式。这种特性对于根据计算结果快速获取表中内容是有用的。

例如，一种进行大小写不敏感比较的常用方法是使用`lower`函数：

```
SELECT * FROM test1 WHERE lower(col1) = 'value';
```

这种查询可以利用一个建立在`lower(col1)`函数结果之上的索引：

```
CREATE INDEX test1_lower_col1_idx ON test1 (lower(col1));
```



如果我们将该索引声明为`UNIQUE`，它将阻止创建在`col1`值上只有大小写不同的行。

另外一个例子，如果我们经常进行如下的查询：

```
SELECT * FROM people WHERE (first_name || ' ' || last_name) = 'John Smith';
```

那么值得创建一个这样的索引：

```
CREATE INDEX people_names ON people ((first_name || ' ' || last_name));
```



正如第二个例子所示，`CREATE INDEX`命令的语法通常要求在被索引的表达式周围书写圆括号。而如第一个例子所示，当表达式只是一个函数调用时可以省略掉圆括号。

索引表达式的维护代价较为昂贵，因为在每一个行被插入或更新时都得为它重新计算相应的表达式。然而，索引表达式在进行索引搜索时却***不\***需要重新计算，因为它们的结果已经被存储在索引中了。在上面两个例子中，系统将会发现查询的条件是`WHERE indexedcolumn = 'constant'`，因此查询的速度将等同于其他简单索引查询。因此，表达式索引对于检索速度远比插入和更新速度重要的情况非常有用。

### 部分索引

一个*部分索引*是建立在表的一个子集上，而该子集则由一个条件表达式（被称为部分索引的*谓词*）定义。而索引中只包含那些符合该谓词的表行的项。部分索引是一种专门的特性，但在很多种情况下它们也很有用。

使用部分索引的一个主要原因是避免索引公值。由于搜索一个公值的查询（一个在所有表行中占比超过一定百分比的值）不会使用索引，所以完全没有理由将这些行保留在索引中。这可以减小索引的尺寸，同时也将加速使用索引的查询。它也将加速很多表更新操作，因为这种索引并不需要在所有情况下都被更新

**Example 11.1. 建立一个部分索引来排除公值**

假设我们要在一个数据库中保存网页服务器访问日志。大部分访问都来自于我们组织内的IP地址，但是有些来自于其他地方（如使用拨号连接的员工）。如果我们主要通过IP搜索来自于外部的访问，我们就没有必要索引对应于我们组织内网的IP范围。

假设有这样一个表：

```
CREATE TABLE access_log (
    url varchar,
    client_ip inet,
    ...
);
```



用以下命令可以创建适用于我们的部分索引：

```
CREATE INDEX access_log_client_ip_ix ON access_log (client_ip)
WHERE NOT (client_ip > inet '192.168.100.0' AND
           client_ip < inet '192.168.100.255');
```



一个使用该索引的典型查询是：

```
SELECT *
FROM access_log
WHERE url = '/index.html' AND client_ip = inet '212.78.10.32';
```

一个不能使用该索引的查询：

```
SELECT *
FROM access_log
WHERE client_ip = inet '192.168.100.23';
```

可以看到部分索引查询要求公值能被预知，因此部分索引最适合于数据分布不会改变的情况。当然索引也可以偶尔被重建来适应新的数据分布，但是这会增加维护负担。

**Example 11.2. 建立一个部分索引来排除不感兴趣的值**

如果我们有一个表包含已上账和未上账的订单，其中未上账的订单在整个表中占据一小部分且它们是最经常被访问的行。我们可以通过只在未上账的行上创建一个索引来提高性能。创建索引的命令如下：

```
CREATE INDEX orders_unbilled_index ON orders (order_nr)
    WHERE billed is not true;
```



使用该索引的一个可能查询是：

```
SELECT * FROM orders WHERE billed is not true AND order_nr < 10000;
```

然而，索引也可以用于完全不涉及`order_nr`的查询，例如：

```
SELECT * FROM orders WHERE billed is not true AND amount > 5000.00;
```

这并不如在`amount`列上部分索引有效，因为系统必须扫描整个索引。然而，如果有相对较少的未上账订单，使用这个部分索引来查找未上账订单将会更好。

注意这个查询将不会使用该索引：

```
SELECT * FROM orders WHERE order_nr = 3501;
```

订单3501可能在已上账订单或未上账订单中。

PostgreSQL支持使用任意谓词的部分索引，只要其中涉及的只有被索引表的列。然而，记住谓词必须匹配在将要受益于索引的查询中使用的条件。更准确地，只有当系统能识别查询的`WHERE`条件从数学上索引的谓词时，一个部分索引才能被用于一个查询。PostgreSQL并不能给出一个精致的定理证明器来识别写成不同形式在数学上等价的表达式（一方面创建这种证明器极端困难，另一方面即便能创建出来对于实用也过慢）。系统可以识别简单的不等蕴含，例如“x < 1”蕴含“x < 2”；否则谓词条件必须准确匹配查询的`WHERE`条件中的部分，或者索引将不会被识别为可用。

> PostgreSQL并不能给出一个精致的定理证明器来识别写成不同形式在数学上等价的表达式（一方面创建这种证明器极端困难，另一方面即便能创建出来对于实用也过慢）

**Example 11.3. 建立一个部分唯一索引**

假设我们有一个描述测试结果的表。我们希望保证其中对于一个给定的主题和目标组合只有一个“成功”项，但其中可能会有任意多个“不成功”项。实现它的方式是：

```
CREATE TABLE tests (
    subject text,
    target text,
    success boolean,
    ...
);

CREATE UNIQUE INDEX tests_success_constraint ON tests (subject, target)
    WHERE success;
```

当有少数成功测试和很多不成功测试时这是一种特别有效的方法。

记住建立一个部分索引意味着我们知道的至少和查询规划器所知的一样多，尤其是我们知道什么时候一个索引会是有益的。构建这些知识需要经验和对于PostgreSQL中索引工作方式的理解。在大部分情况下，一个部分索引相对于一个普通索引的优势很小。

### 操作符类和操作符族

一个索引定义可以为索引中的每一列都指定一个*操作符类*。

```
CREATE INDEX name ON table (column opclass [sort options] [, ...]);
```

### 索引优化总结

1. gin 索引 & pg_trgm 模块

   Postgres使用trigram将字符串分解成更小的单元便于有效地索引它们。pg_trgm模块支持GIST或GIN索引，从9.1开始，这些索引支持LIKE/ILIKE查询。

   要使用pg_trgm模块，首先要启用该扩展，然后使用`gin_trgm_ops`创建索引

   > pg_trgm模块提供函数和操作符测定字母，数字，文本基于三元模型匹配的相似性， 还有支持快速搜索相似字符串的索引操作符类。

```sql
 CREATE INDEX idx_users_username ON users (username);
 
 EXPLAIN ANALYSE SELECT COUNT(*) FROM users WHERE username ILIKE '%foo%';
 
 CREATE EXTENSION pg_trgm;
 CREATE INDEX trgm_idx_users_username ON users USING gin (username gin_trgm_ops);
 
 EXPLAIN ANALYSE SELECT COUNT(*) FROM users WHERE username ILIKE '%foo%';
 
 --联合索引
 CREATE INDEX index_users_full_name ON users using gin ((first_name || ' ' || last_name) gin_trgm_ops);
 
 EXPLAIN ANALYSE SELECT COUNT(*) FROM users WHERE first_name || ' ' || last_name ILIKE '%foo%';
```

[gin&pg_trgm](https://razeencheng.com/post/pg-like-index-optimize.html)

2. 表达式索引是一个比较灵活的，比如可以对一个text字段的前6个字符建立索引。

```sql
CREATE INDEX blocks_blockNumber ON blocks (LEFT(blockHash,6))
```

3. vaccum 命令能够释放被删除元组的磁盘空间。
4. CLUSTER  CLUSTER — 根据一个索引聚簇一个表，由于一个表只能有一个物理顺序，所以每个表只能有一个聚集索引，并且应该仔细选择要用于聚集的索引。`Unlike Microsoft SQL Server, clustering on an index in PostgreSQL does not maintain that order. You have to reapply the CLUSTER process to maintain the order.`每次数据更改后需要重新使用cluster建立集群。对于cluster写入先写入每一页的`FILLFACTOR`的空间，然后再写入新的页面。cluster不能再部分索引上建立集群。

```SQL 
CREATE INDEX transactions_400W_500W_blockNumber ON transactions_400W_500W (blockNumber) WHERE NOT blockNumber=0;
CLUSTER transactions_400W_500W USING transactions_400W_500W_blockNumber;
ERROR:  cannot cluster on partial index "transactions_600w_700w_blocknumber"
```

**tip**

- 注意在cluster时，盘簇化是一次性操作：当表将来被更新之后，更改的内容不会被盘簇化排序
- 在对一个表进行盘簇化排序的时候，会在其上请求一个 ACCESS EXCLUSIVE 锁，其它客户端即不能读也不能写
- 磁盘空间会需要至少约 2 倍的表大小和索引大小
- 在执行任何操作之前，必须初始化磁盘上的数据库存储区域。我们称其为*数据库集群*。（SQL标准使用术语目录集群。）数据库集群是由运行中的数据库服务器的单个实例管理的数据库的集合。初始化之后，数据库集群将包含一个名为的数据库`postgres`，这是公用程序，用户和第三方应用程序使用的默认数据库。数据库服务器本身不需要`postgres`数据库存在，但是许多外部实用程序都假定它存在。初始化期间在每个集群内创建的另一个数据库称为`template1`。顾名思义，它将用作后续创建的数据库的模板；它不应该用于实际工作。

> cluster也有数据库集群的功能，也就是一个新的数据库数据目录。

**Syntax of Cluster:**
First time you must execute CLUSTER using the Index Name.

`CLUSTER table_name USING index_name;`

**Cluster the table:**
Once you have executed CLUSTER with Index, next time you should execute only CLUSTER TABLE because It knows that which index already defined as CLUSTER.

  `CLUSTER table_name;`

**Cluster all tables of database:**

`  CLUSTER;`