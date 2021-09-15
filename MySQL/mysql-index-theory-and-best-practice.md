# 一文读懂MySQL的索引结构及查询优化

回顾前文: [一文学会MySQL的explain工具](https://www.cnblogs.com/itwild/p/13424113.html)

(同时再次强调，这几篇关于MySQL的探究都是基于`5.7`版本，相关总结与结论`不一定适用`于其他版本)

MySQL官方文档中(`https://dev.mysql.com/doc/refman/5.7/en/optimization-indexes.html`)有这样一段描述：

> The best way to improve the performance of SELECT operations is to create indexes on one or more of the columns that are tested in the query. But unnecessary indexes waste space and waste time for MySQL to determine which indexes to use. Indexes also add to the cost of inserts, updates, and deletes because each index must be updated. You must find the right balance to achieve fast queries using the optimal set of indexes.

就是说提高查询性能最直接有效的方法就是建立索引，但是不必要的索引会浪费空间，同时也增加了额外的时间成本去判断应该走哪个索引，此外，索引还会增加插入、更新、删除数据的成本，因为做这些操作的同时还要去维护(更新)索引树。因此，应该学会使用最佳索引集来优化查询。

## 索引结构

参考：
1. 《MySQL索引背后的数据结构及算法原理》`http://blog.codinglabs.org/articles/theory-of-mysql-index.html`

2. 《Mysql BTree和B+Tree详解》`https://www.cnblogs.com/Transkai/p/11595405.html`

3. 《为什么MySQL使用B+树》`https://draveness.me/whys-the-design-mysql-b-plus-tree/`

4. 《浅入浅出MySQL和InnoDB》`https://draveness.me/mysql-innodb/`

5. 《漫画：什么是B树？》`https://mp.weixin.qq.com/s/rDCEFzoKHIjyHfI_bsz5Rw`

### 什么是索引

在MySQL中，索引(`Index`)是帮助高效获取数据的数据结构。这种数据结构MySQL中最常用的就是B+树(`B+Tree`)。

> Indexes are used to find rows with specific column values quickly. Without an index, MySQL must begin with the first row and then read through the entire table to find the relevant rows.

就好比给你一本书和一篇文章标题，如果没有目录，让你找此标题对应的文章，可能需要从第一页翻到最后一页；如果有目录大纲，你可能只需要在目录页寻找此标题，然后迅速定位文章。

这里我们可以把`书(book)`看成是MySQL中的`table`，把`文章(article)`看成是`table`中的一行记录，即`row`，`文章标题(title)`看成`row`中的一列`column`，`目录`自然就是对`title`列建立的索引`index`了，这样`根据文章标题从书中检索文章`就对应sql语句`select * from book where title = ?`，相应的，书中每增加一篇文章(即`insert into book (title, ...) values ('华山论剑', ...)`)，都需要维护一下`目录`，这样才能从目录中找到新增的文章`华山论剑`，这一操作对应的是MySQL中每插入(`insert`)一条记录需要维护`title`列的索引树(`B+Tree`)。

### 为什么使用B+Tree

首先需要澄清的一点是，MySQL跟B+树没有直接的关系，真正与B+树有关系的是MySQL的默认存储引擎`InnoDB`，MySQL中存储引擎的主要作用是`负责数据的存储和提取`，除了`InnoDB`之外，MySQL中也支持比如`MyISAM`等其他存储引擎(详情见`https://dev.mysql.com/doc/refman/5.7/en/storage-engine-setting.html`)作为表的底层存储引擎。

```sh
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
```
提到索引，我们可能会立马想到下面几种数据结构来实现。

(1) 哈希表  
哈希虽然能够提供`O(1)`的单数据行的查询性能，但是对于`范围查询`和`排序`却无法很好支持，需全表扫描。

(2) 红黑树  
红黑树(`Red Black Tree`)是一种自平衡二叉查找树，在进行插入和删除操作时通过特定操作保持二叉查找树的平衡，从而获得较高的查找性能。

一般来说，索引本身也很大，往往不可能全部存储在内存中，因此索引往往以索引文件的形式存储的磁盘上。这样的话，索引查找过程中就要产生磁盘I/O消耗，相对于内存存取，I/O存取的消耗远远高于内存，所以评价一个数据结构作为索引的优劣最重要的指标就是查找过程中磁盘I/O次数。换句话说，`索引的结构组织要尽量减少查找过程中磁盘I/O的次数。`

在这里，磁盘I/O的次数取决于树的高度，所以，在数据量较大时，`红黑树会因树的高度较大而造成磁盘IO较多`，从而影响查询效率。

(3) B-Tree  
B树中的B代表平衡(`Balance`)，而不是二叉(`Binary`)，B树是从平衡二叉树演化而来的。

为了降低树的高度(也就是减少磁盘I/O次数)，把原来`瘦高`的树结构变得`矮胖`，B树会在`每个节点存储多个元素`(红黑树每个节点只会存储一个元素)，并且节点中的元素从左到右递增排列。如下图所示：

![B-Tree结构图](https://img2020.cnblogs.com/blog/1546632/202008/1546632-20200830195348368-1304078258.png)

`B-Tree`在查询的时候比较次数其实不比二叉查找树少，但在内存中的大小比较、二分查找的耗时相比磁盘IO耗时几乎可以忽略。 `B-Tree大大降低了树的高度`，所以也就极大地提升了查找性能。

(4) B+Tree  
`B+Tree`是在`B-Tree`基础上进一步优化，使其更适合实现存储索引结构。InnoDB存储引擎就是用`B+Tree`实现其索引结构。

`B-Tree`结构图中可以看到每个节点中不仅包含数据的`key`值，还有`data`值。而每一个节点的存储空间是有限的，如果`data`值较大时将会导致每个节点能存储的`key`的数量很小，这样会导致B-Tree的高度变大，增加了查询时的磁盘I/O次数，进而影响查询性能。在`B+Tree`中，所有`data`值都是按照键值大小顺序存放在同一层的叶子节点上，而`非叶子节点上只存储key值信息`，这样可以增大每个非叶子节点存储的`key`值数量，降低B+Tree的高度，提高效率。

![B+Tree结构图](https://img2020.cnblogs.com/blog/1546632/202008/1546632-20200830201413134-394816073.png)

**这里补充一点相关知识** 在计算机中，磁盘往往不是严格按需读取，而是每次都会预读，即使只需要一个字节，磁盘也会从这个位置开始，顺序向后读取一定长度的数据放入内存。这样做的理论依据是计算机科学中著名的`局部性原理`：

> 当一个数据被用到时，其附近的数据也通常会马上被使用。

由于磁盘顺序读取的效率很高（不需要寻道时间，只需很少的旋转时间），因此对于具有局部性的程序来说，预读可以提高I/O效率。预读的长度一般为页(`page`)的整数倍。

`页`是计算机管理存储器的逻辑块，硬件及操作系统往往将主存和磁盘存储区分割为连续的大小相等的块，每个存储块称为一页(许多操作系统的页默认大小为`4KB`)，主存和磁盘以页为单位交换数据。当程序要读取的数据不在主存中时，会触发一个缺页异常，此时操作系统会向磁盘发出读盘信号，磁盘会找到数据的起始位置并向后连续读取一页或几页载入内存中，然后异常返回，程序继续运行。(如下命令可以查看操作系统的默认页大小)
```sh
$ getconf PAGE_SIZE
4096
```
数据库系统的设计者巧妙利用了磁盘预读原理，将一个节点的大小设为操作系统的页大小的整数倍，这样每个节点只需要一次I/O就可以完全载入。

`InnoDB`存储引擎中也有页(`Page`)的概念，页是其磁盘管理的最小单位。InnoDB存储引擎中默认每个页的大小为16KB。
```sh
mysql> show variables like 'innodb_page_size';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| innodb_page_size | 16384 |
+------------------+-------+
1 row in set (0.01 sec)
```
一般表的主键类型为`INT`（占4个字节）或`BIGINT`（占8个字节），指针类型也一般为4或8个字节，也就是说一个页（B+Tree中的一个节点）中大概存储`16KB/(8B+8B)=1K`个键值（因为是估值，为方便计算，这里的K取值为`10^3`）。也就是说一个深度为3的B+Tree索引可以维护`10^3 * 10^3 * 10^3 = 10亿`条记录。

`B+Tree`的高度一般都在2到4层。mysql的InnoDB存储引擎在设计时是将根节点常驻内存的，也就是说查找某一键值的行记录时最多只需要1到3次磁盘I/O操作。

随机I/O对于MySQL的查询性能影响会非常大，而顺序读取磁盘中的数据会很快，由此我们也应该尽量减少随机I/O的次数，这样才能提高性能。在`B-Tree`中由于所有的节点都可能包含目标数据，我们总是要从根节点向下遍历子树查找满足条件的数据行，这会带来大量的随机I/O，而`B+Tree`所有的数据行都存储在叶子节点中，而这些叶子节点通过`双向链表`依次按顺序连接，当我们在B+树遍历数据(比如说`范围查询`)时可以直接在多个叶子节点之间进行跳转，保证`顺序`、`倒序`遍历的性能。

另外，对以上提到的数据结构不熟悉的朋友，这里推荐一个在线数据结构可视化演示工具，有助于快速理解这些数据结构的机制：`https://www.cs.usfca.edu/~galles/visualization/Algorithms.html`
### 主键索引

上面也有提及，在MySQL中，索引属于存储引擎级别的概念。不同存储引擎对索引的实现方式是不同的，这里主要看下`MyISAM`和`InnoDB`两种存储引擎的索引实现方式。

#### MyISAM索引实现

`MyISAM`引擎使用`B+Tree`作为索引结构时叶子节点的`data`域存放的是数据记录的地址。如下图所示：

![MyISAM主键索引原理图](https://img2020.cnblogs.com/blog/1546632/202009/1546632-20200919221036646-2109444544.png)

由上图可以看出：MyISAM索引文件和数据文件是分离的，索引文件仅保存数据记录的地址，因此MyISAM的索引方式也叫做`非聚集`的，之所以这么称呼是为了与InnoDB的`聚集索引`区分。

#### InnoDB索引实现

`InnoDB`的`主键索引`也使用`B+Tree`作为索引结构时的实现方式却与MyISAM截然不同。`InnoDB的数据文件本身就是索引文件`。在InnoDB中，表数据文件本身就是按`B+Tree`组织的一个索引结构，这棵树的叶子节点`data`域保存了完整的数据记录，这个索引的`key`是数据表的主键，因此InnoDB表数据文件本身就是主索引。

![InnoDB主键索引原理图](https://img2020.cnblogs.com/blog/1546632/202009/1546632-20200919222451055-995072120.png)

`InnoDB`存储引擎中的主键索引(`primary key`)又叫做聚集索引(`clustered index`)。因为InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。(详情见官方文档：`https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html`)

聚集索引这种实现方式使得按主键搜索十分高效，直接能查出整行数据。

在InnoDB中，用非单调递增的字段作为主键不是个好主意，因为InnoDB数据文件本身是一棵`B+Tree`，非单增的主键会造成在插入新记录时数据文件为了`维持B+Tree的特性`而频繁的分裂调整，十分低效，因而使用`递增`字段作为主键则是一个很好的选择。

### 非主键索引

#### MyISAM索引实现

MyISAM中，主键索引和非主键索引（`Secondary key`，也有人叫做`辅助索引`）在结构上没有任何区别，只是主键索引要求key是唯一的，而辅助索引的key可以重复。这里不再多加叙述。

#### InnoDB索引实现

InnoDB的非主键索引`data`域存储相应记录`主键的值`。换句话说，InnoDB的所有非主键索引都引用主键的值作为data域。如下图所示：

![InnoDB非主键索引原理图](https://img2020.cnblogs.com/blog/1546632/202009/1546632-20200919225813936-1490118290.png)

由上图可知：使用非主键索引搜索时需要检索两遍索引，首先检索非主键索引获得主键(`primary key`)，然后用主键到`主键索引树`中检索获得完整记录。

那么为什么非主键索引结构叶子节点存储的是主键值，而不像主键索引那样直接存储完整的一行数据，这样就能避免回表二次检索？显然，这样做一方面节省了大量的存储空间，另一方面多份冗余数据，更新数据的效率肯定低下，另外保证数据的一致性是个麻烦事。

到了这里，也很容易明白为什么`不建议使用过长的字段作为主键`，因为所有的非主键索引都引用主键值，过长的主键值会让非主键索引变得过大。

### 联合索引

官方文档：`https://dev.mysql.com/doc/refman/5.7/en/multiple-column-indexes.html`

比如`INDEX idx_book_id_hero_name (book_id, hero_name) USING BTREE`，即对`book_id, hero_name`两列建立了一个联合索引。

> A multiple-column index can be considered a sorted array, the rows of which contain values that are created by concatenating the values of the indexed columns.

联合索引是多列按照次序一列一列比较大小，拿`idx_book_id_hero_name`这个联合索引来说，先比较`book_id`，book_id小的排在左边，book_id大的排在右边，book_id相同时再比较`hero_name`。如下图所示：

![InnoDB联合索引原理图](https://img2020.cnblogs.com/blog/1546632/202009/1546632-20200920111026527-1672463564.png)

了解了联合索引的结构，就能引入`最左前缀法则`：

> If the table has a multiple-column index, any leftmost prefix of the index can be used by the optimizer to look up rows. For example, if you have a three-column index on (col1, col2, col3), you have indexed search capabilities on (col1), (col1, col2), and (col1, col2, col3).

就是说联合索引中的多列是按照列的次序排列的，如果查询的时候不能满足列的次序，比如说where条件中缺少`col1 = ?`，直接就是`col2 = ? and col3 = ?`，那么就走不了联合索引，从上面联合索引的结构图应该能明显看出，只有`col2`列无法通过索引树检索符合条件的数据。

根据最左前缀法则，我们知道对`INDEX idx_book_id_hero_name (book_id, hero_name)`来说，`where book_id = ? and hero_name = ?`的查询来说，肯定可以走索引，但是如果是`where hero_name = ? and book_id = ?`呢，表面上看起来不符合最左前缀法则啊，但MySQL优化器会根据已有的索引，调整查询条件中这两列的顺序，让它符合最左前缀法则，走索引，这里也就回答了上篇《一文学会MySQL的explain工具》中为什么用`show warnings`命令查看时，`where`中的两个过滤条件`hero_name`、`book_id`先后顺序被调换了。

至于对联合索引中的列进行范围查询等各种情况，都可以先想联合索引的结构是如何创建出来的，然后看过滤条件是否满足最左前缀法则。比如说范围查询时，范围列可以用到索引（必须是最左前缀），但是范围列后面的列无法用到索引。同时，索引最多用于一个范围列，因此如果查询条件中有两个范围列则无法全用到索引。

## 优化建议

### 主键的选择

在使用`InnoDB`存储引擎时，如果没有特别的需要，尽量使用一个与业务无关的`递增字段`作为主键，主键字段不宜过长。原因上面在讲索引结构时已提过。比如说常用雪花算法生成64bit大小的整数(占8个字节，用`BIGINT`类型)作为主键就是一个不错的选择。

### 索引的选择

(1) 表记录比较少的时候，比如说只有几百条记录的表，对一些列建立索引的意义可能并不大，所以表记录不大时酌情考虑索引。但是业务上具有`唯一特性`的字段，即使是多个字段的组合，也建议使用唯一索引(`UNIQUE KEY`)。

(2) 当索引的选择性非常低时，索引的意义可能也不大。所谓索引的选择性(`Selectivity`)，是指不重复的索引值(也叫基数`Cardinality`)与表记录数的比值，即`count(distinct 列名)/count(*)`，常见的场景就是有一列`status`标识数据行的状态，可能`status`非0即1，总数据100万行有50万行`status`为0，50万行`status`为1，那么是否有必要对这一列单独建立索引呢？

> An index is best used when you need to select a small number of rows in comparison to the total rows.

这句话我摘自stackoverflow上《MySQL: low selectivity columns = how to index?》下面一个人的回答。(详情见：`https://stackoverflow.com/questions/2386852/mysql-low-cardinality-selectivity-columns-how-to-index`)

对于上面说的`status`非0即1，而且这两种情况分布比较均匀的情况，索引可能并没有实际意义，实际查询时，MySQL优化器在计算全表扫描和索引树扫描代价后，可能会放弃走索引，因为先从`status`索引树中遍历出来主键值，再去主键索引树中查最终数据，代价可能比全表扫描还高。

但是如果对于`status`为1的数据只有1万行，其他99万行数据`status`为0的情况呢，你怎么看？欢迎有兴趣的朋友在文章下面留言讨论！

**补充**: 关于MySQL如何选择走不走索引或者选择走哪个最佳索引，可以使用MySQL自带的trace工具一探究竟。具体使用见下面的官方文档。  
`https://dev.mysql.com/doc/internals/en/optimizer-tracing.html`   
`https://dev.mysql.com/doc/refman/5.7/en/information-schema-optimizer-trace-table.html`

使用方法：
```sh
mysql> set session optimizer_trace="enabled=on",end_markers_in_json=on;
mysql> select * from tb_hero where hero_id = 1;
mysql> SELECT * FROM information_schema.OPTIMIZER_TRACE;
```
`注意`：开启trace工具会影响MySQL性能，所以只能临时分析sql使用，用完之后应当立即关闭
```sh
mysql> set session optimizer_trace="enabled=off";
```

(3) 在`varchar`类型字段上建立索引时，建议指定`索引长度`，有些时候可能没必要对全字段建立索引，根据实际文本区分度决定索引长度即可【说明：索引的长度与区分度是一对矛盾体，`一般对字符串类型数据，长度为20的索引，区分度会高达90%以上`，可以使用`count(distinct left(列名, 索引长度))/count(*)`来确定区分度】。

这种指定索引长度的索引叫做`前缀索引`(详情见`https://dev.mysql.com/doc/refman/5.7/en/column-indexes.html#column-indexes-prefix`)。
> With col_name(N) syntax in an index specification for a string column, you can create an index that uses only the first N characters of the column. Indexing only a prefix of column values in this way can make the index file much smaller. When you index a BLOB or TEXT column, you must specify a prefix length for the index.

前缀索引语法如下：
```sh
mysql> alter table tb_hero add index idx_hero_name_skill2 (hero_name, skill(2));
```
前缀索引兼顾索引大小和查询速度，但是其缺点是不能用于`group by`和`order by`操作，也不能用于`covering index`（即当索引本身包含查询所需全部数据时，不再访问数据文件本身）。

(4) 当查询语句的`where`条件或`group by`、`order by`含多列时，可根据实际情况优先考虑联合索引(`multiple-column index`)，这样可以减少单列索引(`single-column index)`的个数，有助于高效查询。

> If you specify the columns in the right order in the index definition, a single composite index can speed up several kinds of queries on the same table.

建立联合索引时要特别注意`column`的次序，应结合上面提到的`最左前缀法则`以及实际的过滤、分组、排序需求。`区分度最高的建议放最左边`。

说明：
- `order by`的字段可以作为联合索引的一部分，并且放在最后，避免出现`file_sort`的情况，影响查询性能。正例：`where a=? and b=? order by c`会走索引`idx_a_b_c`，但是`WHERE a>10 order by b`却无法完全使用上索引`idx_a_b`，只会使用上联合索引的第一列a

- 存在非等号和等号混合时，在建联合索引时，应该把等号条件的列前置。如：`where c>? and d=?`那么即使c的区分度更高，也应该把d放在索引的最前列，即索引`idx_d_c`

- 如果`where a=? and b=?`，如果a列的几乎接近于唯一值，那么只需要建立单列索引`idx_a`即可

### order by与group by

尽量在索引列上完成分组、排序，遵循索引`最左前缀法则`，如果`order by`的条件不在索引列上，就会产生`Using filesort`，降低查询性能。

### 分页查询

MySQL分页查询大多数写法可能如下：
```sh
mysql> select * from tb_hero limit offset,N;
```
MySQL并不是跳过`offset`行，而是取`offset+N`行，然后返回放弃前`offset`行，返回`N`行，那当`offset`特别大的时候，效率就非常的低下。

可以对超过特定阈值的页数进行SQL改写如下：

先快速定位需要获取的id段，然后再关联
```sh
mysql> select a.* from tb_hero a, (select hero_id from tb_hero where 条件 limit 100000,20 ) b where a.hero_id = b.hero_id;
```
或者这种写法
```sh
mysql> select a.* from tb_hero a inner join (select hero_id from tb_hero where 条件 limit 100000,20) b on a.hero_id = b.hero_id;
```

### 多表join

(1) 需要join的字段，数据类型必须绝对一致；  
(2) 多表join时，保证被关联的字段有索引

### 覆盖索引

利用覆盖索引(`covering index`)来进行查询操作，避免回表，从而增加磁盘I/O。换句话说就是，尽可能避免`select *`语句，只选择必要的列，去除无用的列。

> An index that includes all the columns retrieved by a query. Instead of using the index values as pointers to find the full table rows, the query returns values from the index structure, saving disk I/O. InnoDB can apply this optimization technique to more indexes than MyISAM can, because InnoDB secondary indexes also include the primary key columns. InnoDB cannot apply this technique for queries against tables modified by a transaction, until that transaction ends.

> Any column index or composite index could act as a covering index, given the right query. Design your indexes and queries to take advantage of this optimization technique wherever possible.

当索引本身包含查询所需全部列时，无需回表查询完整的行记录。对于`InnoDB`来说，非主键索引中包含了`所有的索引列`以及`主键值`，查询的时候尽量用这种特性避免回表操作，数据量很大时，查询性能提升很明显。

### in和exsits

原则：`小表驱动大表`，即小的数据集驱动大的数据集

(1) 当A表的数据集大于B表的数据集时，`in`优于`exists`
```sh
mysql> select * from A where id in (select id from B)
```

(2) 当A表的数据集小于B表的数据集时，`exists`优于`in`
```sh
mysql> select * from A where exists (select 1 from B where B.id = A.id)
```

### like

索引文件具有`B+Tree`最左前缀匹配特性，如果左边的值未确定，那么无法使用索引，所以应尽量避免左模糊(即`%xxx`)或者全模糊(即`%xxx%`)。

```sh
mysql> select * from tb_hero where hero_name like '%无%';
+---------+-----------+--------------+---------+
| hero_id | hero_name | skill        | book_id |
+---------+-----------+--------------+---------+
|       3 | 张无忌    | 九阳神功     |       3 |
|       5 | 花无缺    | 移花接玉     |       5 |
+---------+-----------+--------------+---------+
2 rows in set (0.00 sec)

mysql> explain select * from tb_hero where hero_name like '%无%';
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | tb_hero | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    6 |    16.67 | Using where |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
可以看出全模糊查询时全表扫了，这个时候使用`覆盖索引`的特性，只选择索引字段可以有所优化。如下：
```sh
mysql> explain select book_id, hero_name from tb_hero where hero_name like '%无%';
+----+-------------+---------+------------+-------+---------------+-----------------------+---------+------+------+----------+--------------------------+
| id | select_type | table   | partitions | type  | possible_keys | key                   | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+---------+------------+-------+---------------+-----------------------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | tb_hero | NULL       | index | NULL          | idx_book_id_hero_name | 136     | NULL |    6 |    16.67 | Using where; Using index |
+----+-------------+---------+------------+-------+---------------+-----------------------+---------+------+------+----------+--------------------------+
1 row in set, 1 warning (0.00 sec)
```

### count(*)

阿里巴巴Java开发手册中有这样的规约：
> 不要使用`count(列名)`或`count(常量)`来替代`count(*)`，`count(*)`是SQL92定义的标准统计行数的语法，跟数据库无关，跟`NULL`和`非NULL`无关【说明：`count(*)`会统计值为`NULL`的行，而`count(列名)`不会统计此列为`NULL`值的行】。
`count(distinct col)`计算该列除`NULL`之外的不重复行数，注意`count(distinct col1, col2)`如果其中一列全为`NULL`，那么即使另一列有不同的值，也返回为0

截取一段官方文档对`count`的描述(具体见：`https://dev.mysql.com/doc/refman/5.7/en/aggregate-functions.html#function_count`)

> COUNT(expr): Returns a count of the number of non-NULL values of expr in the rows.The result is a BIGINT value.If there are no matching rows, COUNT(expr) returns 0.

> COUNT(*) is somewhat different in that it returns a count of the number of rows, whether or not they contain NULL values.

> Prior to MySQL 5.7.18, InnoDB processes SELECT `COUNT(*)` statements by scanning the clustered index. As of MySQL 5.7.18, InnoDB processes SELECT COUNT(*) statements by traversing the smallest available secondary index unless an index or optimizer hint directs the optimizer to use a different index. If a secondary index is not present, the clustered index is scanned.

可见`5.7.18`之前，MySQL处理`count(*)`会扫描主键索引，`5.7.18`之后从非主键索引中选择较小的合适的索引扫描。可以用`explain`看下执行计划。
```sh
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.18    |
+-----------+
1 row in set (0.00 sec)

mysql> explain select count(*) from tb_hero;
+----+-------------+---------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | tb_hero | NULL       | index | NULL          | idx_skill | 15      | NULL |    6 |   100.00 | Using index |
+----+-------------+---------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select count(1) from tb_hero;
+----+-------------+---------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | tb_hero | NULL       | index | NULL          | idx_skill | 15      | NULL |    6 |   100.00 | Using index |
+----+-------------+---------+------------+-------+---------------+-----------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
有人纠结`count(*)`、`count(1)`到底哪种写法更高效，从上面的执行计划来看都一样，如果你还不放心的话，官方文档中也明确指明了`InnoDB`对`count(*)`、`count(1)`的处理完全一致。
> InnoDB handles SELECT COUNT(*) and SELECT COUNT(1) operations in the same way. There is no performance difference.

### 其他

索引列上做任何操作(`表达式`、`函数计算`、`类型转换`等)时无法使用索引会导致全表扫描

## 实战

前几周测试同事对公司的某产品进行压测，某单表写入了近2亿条数据，过程中发现配的报表有几个数据查询时间太长，所以重点看了几个慢查询SQL。避免敏感信息，这里对其提取简化做个记录。
```sh
mysql> select count(*) from tb_alert;
+-----------+
| count(*)  |
+-----------+
| 198101877 |
+-----------+
```

### 表join慢

表join后，取前10条数据就花了15秒，看了下SQL执行计划，如下：

```sh
mysql> select * from tb_alert left join tb_situation_alert on tb_alert.alert_id = tb_situation_alert.alert_id limit 10;
10 rows in set (15.46 sec)

mysql> explain select * from tb_alert left join tb_situation_alert on tb_alert.alert_id = tb_situation_alert.alert_id limit 10;
+----+-------------+--------------------+------------+------+---------------+------+---------+------+-----------+----------+----------------------------------------------------+
| id | select_type | table              | partitions | type | possible_keys | key  | key_len | ref  | rows      | filtered | Extra                                              |
+----+-------------+--------------------+------------+------+---------------+------+---------+------+-----------+----------+----------------------------------------------------+
|  1 | SIMPLE      | tb_alert           | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 190097118 |   100.00 | NULL                                               |
|  1 | SIMPLE      | tb_situation_alert | NULL       | ALL  | NULL          | NULL | NULL    | NULL |   8026988 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+--------------------+------------+------+---------------+------+---------+------+-----------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.00 sec)
```

可以看出join的时候没有用上索引，`tb_situation_alert`表上`联合主键`是这样的`PRIMARY KEY (situation_id, alert_id)`，参与表join字段是`alert_id`，原来是不符合联合索引的最左前缀法则，仅从这条sql看，解决方案有两种，一种是对`tb_situation_alert`表上的`alert_id`单独建立索引，另外一种是调换联合主键的列的次序，改为`PRIMARY KEY (alert_id, situation_id)`。当然不能因为多配一张报表，就改其他产线的表的主键索引，这并不合理。在这里，应该对`alert_id`列单独建立索引。

```sh
mysql> create index idx_alert_id on tb_situation_alert (alert_id);

mysql> select * from tb_alert left join tb_situation_alert on tb_alert.alert_id = tb_situation_alert.alert_id limit 100;
100 rows in set (0.01 sec)

mysql> explain select * from tb_alert left join tb_situation_alert on tb_alert.alert_id = tb_situation_alert.alert_id limit 100;
+----+-------------+--------------------+------------+------+---------------+--------------+---------+---------------------------------+-----------+----------+-------+
| id | select_type | table              | partitions | type | possible_keys | key          | key_len | ref                             | rows      | filtered | Extra |
+----+-------------+--------------------+------------+------+---------------+--------------+---------+---------------------------------+-----------+----------+-------+
|  1 | SIMPLE      | tb_alert           | NULL       | ALL  | NULL          | NULL         | NULL    | NULL                            | 190097118 |   100.00 | NULL  |
|  1 | SIMPLE      | tb_situation_alert | NULL       | ref  | idx_alert_id  | idx_alert_id | 8       | tb_alert.alert_id |         2 |   100.00 | NULL  |
+----+-------------+--------------------+------------+------+---------------+--------------+---------+---------------------------------+-----------+----------+-------+
2 rows in set, 1 warning (0.00 sec)
```
优化后，执行计划可以看出join的时候走了索引，查询前100条0.01秒，和之前的取前10条数据就花了15秒天壤之别。

### 分页查询慢

从第10000000条数据往后翻页时，25秒才能出结果，这里就能使用上面的分页查询优化技巧了。上面讲优化建议时，没看执行计划，这里正好看一下。

```sh
mysql> select * from tb_alert limit 10000000, 10;
10 rows in set (25.23 sec)

mysql> explain select * from tb_alert limit 10000000, 10;
+----+-------------+----------+------------+------+---------------+------+---------+------+-----------+----------+-------+
| id | select_type | table    | partitions | type | possible_keys | key  | key_len | ref  | rows      | filtered | Extra |
+----+-------------+----------+------------+------+---------------+------+---------+------+-----------+----------+-------+
|  1 | SIMPLE      | tb_alert | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 190097118 |   100.00 | NULL  |
+----+-------------+----------+------------+------+---------------+------+---------+------+-----------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```
再看下使用上分页查询优化技巧的sql的执行计划
```sh
mysql> select * from tb_alert a inner join (select alert_id from tb_alert limit 10000000, 10) b on a.alert_id = b.alert_id;
10 rows in set (2.29 sec)

mysql> explain select * from tb_alert a inner join (select alert_id from tb_alert a2 limit 10000000, 10) b on a.alert_id = b.alert_id;
+----+-------------+------------+------------+--------+---------------+---------------+---------+-----------+-----------+----------+-------------+
| id | select_type | table      | partitions | type   | possible_keys | key           | key_len | ref       | rows      | filtered | Extra       |
+----+-------------+------------+------------+--------+---------------+---------------+---------+-----------+-----------+----------+-------------+
|  1 | PRIMARY     | <derived2> | NULL       | ALL    | NULL          | NULL          | NULL    | NULL      |  10000010 |   100.00 | NULL        |
|  1 | PRIMARY     | a          | NULL       | eq_ref | PRIMARY       | PRIMARY       | 8       | b.alert_id |         1 |   100.00 | NULL        |
|  2 | DERIVED     | a2         | NULL       | index  | NULL          | idx_processed | 5       | NULL      | 190097118 |   100.00 | Using index |
+----+-------------+------------+------------+--------+---------------+---------------+---------+-----------+-----------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)
```

### 分组聚合慢

分析SQL后，发现根本上并非分组聚合慢，而是扫描联合索引后，回表导致性能低下，去除不必要的字段，使用覆盖索引。

这里避免敏感信息，只演示分组聚合前的简化SQL，主要问题也是在这。
表上有联合索引`KEY idx_alert_start_host_template_id ( alert_start, alert_host, template_id)`，优化前的sql为
```sh
mysql> select alert_start, alert_host, template_id, alert_service from tb_alert where alert_start > {ts '2019-06-05 00:00:10.0'} limit 10000;
10000 rows in set (1 min 5.22 sec)
```
使用覆盖索引，去掉`alert_service`列，就能避免回表，查询时间从1min多变为0.03秒，如下：
```sh
mysql> select alert_start, alert_host, template_id from tb_alert where alert_start > {ts '2019-06-05 00:00:10.0'} limit 10000;
10000 rows in set (0.03 sec)

mysql> explain select alert_start, alert_host, template_id from tb_alert where alert_start > {ts '2019-06-05 00:00:10.0'} limit 10000;
+----+-------------+----------+------------+-------+------------------------------------+------------------------------------+---------+------+----------+----------+--------------------------+
| id | select_type | table    | partitions | type  | possible_keys                      | key                                | key_len | ref  | rows     | filtered | Extra                    |
+----+-------------+----------+------------+-------+------------------------------------+------------------------------------+---------+------+----------+----------+--------------------------+
|  1 | SIMPLE      | tb_alert | NULL       | range | idx_alert_start_host_template_id   | idx_alert_start_host_template_id   | 9       | NULL | 95048559 |   100.00 | Using where; Using index |
+----+-------------+----------+------------+-------+------------------------------------+------------------------------------+---------+------+----------+----------+--------------------------+
1 row in set, 1 warning (0.01 sec)
```

## 总结

任何不考虑应用场景的设计都不是最好的设计，就比如说表结构的设计、索引的创建，都应该权衡数据量大小、查询需求、数据更新频率等。  
另外正如`《阿里巴巴java开发手册》`中提到的`索引规约`(详情见：[《Java开发手册》之"异常处理、MySQL 数据库"](https://www.cnblogs.com/itwild/p/12353164.html))：  `创建索引时避免有如下极端误解:`
> 1）宁滥勿缺。认为一个查询就需要建一个索引  
> 2）宁缺勿滥。认为索引会消耗空间、严重拖慢记录的更新以及行的新增速度