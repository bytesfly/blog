# 一文学会MySQL的explain工具

## 开篇说明

(1) 本文将细致介绍MySQL的explain工具，是下一篇《一文读懂MySQL的索引机制及查询优化》的准备篇。

(2) 本文主要基于MySQL`5.7`版本(`https://dev.mysql.com/doc/refman/5.7/en/`)，MySQL`8.x`版本可另行翻阅对应版本文档(`https://dev.mysql.com/doc/refman/8.0/en/`)。

(3) 演示过程中的建库、建表、建索引等语句仅为了测试explain工具的使用，并未考虑实际应用场景的合理性。

## explain工具介绍

相关文档：  
`https://dev.mysql.com/doc/refman/5.7/en/explain.html`  
`https://dev.mysql.com/doc/refman/5.7/en/using-explain.html`

> EXPLAIN is used to obtain a query execution plan (that is, an explanation of how MySQL would execute a query).

简单翻译一下，就是explain用于获取查询执行计划（即MySQL是如何执行一个查询的）。

工作中，我们会遇到慢查询，这个时候我们就可以在`select`语句之前增加`explain`关键字，模拟MySQL优化器执行SQL语句，从而分析该SQL语句有没有用上索引、是否全表扫描、能否进一步优化等。

还是来个快速入门的案例比较直观，依次在mysql的命令行执行下面几条语句(建库、建表sql脚本见下面的`数据准备`部分)：
```sh
mysql> use `explain_test`;
mysql> select * from tb_hero where hero_name = '李寻欢' and book_id = 1;
mysql> explain select * from tb_hero where hero_name = '李寻欢' and book_id = 1;
mysql> show warnings \G
```
得到下面的输出：

```sh
mysql> use `explain_test`;
Database changed
mysql> select * from tb_hero where hero_name = '李寻欢' and book_id = 1;
+---------+-----------+--------------+---------+
| hero_id | hero_name | skill        | book_id |
+---------+-----------+--------------+---------+
|       1 | 李寻欢    | 小李飞刀     |       1 |
+---------+-----------+--------------+---------+
1 row in set (0.00 sec)

mysql> explain select * from tb_hero where hero_name = '李寻欢' and book_id = 1;
+----+-------------+---------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-------+
| id | select_type | table   | partitions | type | possible_keys         | key                   | key_len | ref         | rows | filtered | Extra |
+----+-------------+---------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-------+
|  1 | SIMPLE      | tb_hero | NULL       | ref  | idx_book_id_hero_name | idx_book_id_hero_name | 136     | const,const |    1 |   100.00 | NULL  |
+----+-------------+---------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `explain_test`.`tb_hero`.`hero_id` AS `hero_id`,`explain_test`.`tb_hero`.`hero_name` AS `hero_name`,`explain_test`.`tb_hero`.`skill` AS `skill`,`explain_test`.`tb_hero`.`book_id` AS `book_id` from `explain_test`.`tb_hero` where ((`explain_test`.`tb_hero`.`book_id` = 1) and (`explain_test`.`tb_hero`.`hero_name` = '李寻欢'))
1 row in set (0.00 sec)
```

先别急`explain`语句输出结果每一列表示什么意思(后面会具体描述)，用`show warnings`命令可以得到优化后的查询语句大致长什么样子。

补充：
- 有关`show warnings`更详细的使用见`https://dev.mysql.com/doc/refman/5.7/en/show-warnings.html`
- 有关获取`explain`额外的输出信息见`https://dev.mysql.com/doc/refman/5.7/en/explain-extended.html`

原SQL语句：
```sh
select * from tb_hero where hero_name = '李寻欢' and book_id = 1;
```
优化后的SQL语句：
```sh
select `explain_test`.`tb_hero`.`hero_id`   AS `hero_id`,
       `explain_test`.`tb_hero`.`hero_name` AS `hero_name`,
       `explain_test`.`tb_hero`.`skill`     AS `skill`,
       `explain_test`.`tb_hero`.`book_id`   AS `book_id`
from `explain_test`.`tb_hero`
where ((`explain_test`.`tb_hero`.`book_id` = 1) and (`explain_test`.`tb_hero`.`hero_name` = '李寻欢'))
```
可以看出，MySQL优化器把`*`优化成具体的列名，另外把我`where`中的两个过滤条件`hero_name`、`book_id`先后顺序调换了一下，这种顺序调换是概率性事件还是另有文章？
(哈哈哈，(●´ω｀●)留个悬念，本篇仅介绍explain工具，读了下篇《一文读懂MySQL的索引机制及查询优化》后自然豁然开朗)

## 数据准备

为了方便演示explain工具的使用以及输出结果的含义，准备了一些测试数据，初始化sql脚本如下:

```sh
-- ----------------------------
--  create database
-- ----------------------------
DROP database IF EXISTS `explain_test`;
create database `explain_test` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- switch database
use `explain_test`;

-- ----------------------------
--  table structure for `tb_book`
-- ----------------------------
DROP TABLE IF EXISTS `tb_book`;
CREATE TABLE `tb_book` (
  `book_id` int(11) NOT NULL,
  `book_name` varchar(64) DEFAULT NULL,
  `author` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`book_id`),
  UNIQUE KEY `uk_book_name` (`book_name`) USING BTREE,
  INDEX `idx_author` (`author`) USING BTREE
) ENGINE = InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

BEGIN;
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (1, '多情剑客无情剑', '古龙');
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (2, '笑傲江湖', '金庸');
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (3, '倚天屠龙记', '金庸');
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (4, '射雕英雄传', '金庸');
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (5, '绝代双骄', '古龙');
COMMIT;

-- ----------------------------
--  table structure for `tb_hero`
-- ----------------------------
DROP TABLE IF EXISTS `tb_hero`;
CREATE TABLE `tb_hero` (
  `hero_id` int(11) NOT NULL,
  `hero_name` varchar(32) DEFAULT NULL,
  `skill` varchar(64) DEFAULT NULL,
  `book_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`hero_id`),
  INDEX `idx_book_id_hero_name`(`book_id`, `hero_name`) USING BTREE
) ENGINE = InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

BEGIN;
INSERT INTO `tb_hero`(`hero_id`, `hero_name`, `skill`, `book_id`) VALUES (1, '李寻欢', '小李飞刀', 1);
INSERT INTO `tb_hero`(`hero_id`, `hero_name`, `skill`, `book_id`) VALUES (2, '令狐冲', '独孤九剑', 2);
INSERT INTO `tb_hero`(`hero_id`, `hero_name`, `skill`, `book_id`) VALUES (3, '张无忌', '九阳神功', 3);
INSERT INTO `tb_hero`(`hero_id`, `hero_name`, `skill`, `book_id`) VALUES (4, '郭靖', '降龙十八掌', 4);
INSERT INTO `tb_hero`(`hero_id`, `hero_name`, `skill`, `book_id`) VALUES (5, '花无缺', '移花接玉', 5);
INSERT INTO `tb_hero`(`hero_id`, `hero_name`, `skill`, `book_id`) VALUES (6, '任我行', '吸星大法', 2);
COMMIT;

-- ----------------------------
--  Table structure for `tb_book_hero`
-- ----------------------------
DROP TABLE IF EXISTS `tb_book_hero`;
CREATE TABLE `tb_book_hero` (
  `book_id` int(11) NOT NULL,
  `hero_id` int(11) NOT NULL,
  `user_comment` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`book_id`, `hero_id`) USING BTREE
) ENGINE = InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

BEGIN;
INSERT INTO `tb_book_hero`(`book_id`, `hero_id`, `user_comment`) VALUES (1, 1, '小李飞刀，例无虚发，夺魂索命，弹指之间');
INSERT INTO `tb_book_hero`(`book_id`, `hero_id`, `user_comment`) VALUES (2, 2, '令狐少侠留步!');
INSERT INTO `tb_book_hero`(`book_id`, `hero_id`, `user_comment`) VALUES (3, 3, '尝遍世间善恶，归来仍是少年');
INSERT INTO `tb_book_hero`(`book_id`, `hero_id`, `user_comment`) VALUES (4, 4, '我只要我的靖哥哥!');
INSERT INTO `tb_book_hero`(`book_id`, `hero_id`, `user_comment`) VALUES (5, 5, '风采儒雅亦坦荡，武艺精深兼明智。');
INSERT INTO `tb_book_hero`(`book_id`, `hero_id`, `user_comment`) VALUES (2, 6, '有人就有恩怨，有恩怨就有江湖，人心即是江湖，你如何退出！');
COMMIT;
```

## explain的输出结果

相关文档：   
`https://dev.mysql.com/doc/refman/5.7/en/explain-output.html`

看一下官方文档显示的关于explain输出结果列(`explain output columns`)的含义：

|Column	|JSON Name|	Meaning|
|--|--|--|
|id|	select_id	|The SELECT identifier
|select_type|	None|	The SELECT type|
|table|	table_name|	The table for the output row|
|partitions|	partitions|	The matching partitions|
|type|	access_type|	The join type|
|possible_keys|	possible_keys|	The possible indexes to choose|
|key|	key|	The index actually chosen|
|key_len|	key_length|	The length of the chosen key|
|ref|	ref|	The columns compared to the index|
|rows|	rows|	Estimate of rows to be examined|
|filtered|	filtered|	Percentage of rows filtered by table condition|
|Extra|	None|	Additional information|

其中`JSON Name`指的是当设定`FORMAT=JSON`时，列名在json中显示的name，见下面的演示就明白了
```sh
mysql> explain select * from tb_book \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_book
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 5
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)

mysql> explain FORMAT=JSON select * from tb_book \G
*************************** 1. row ***************************
EXPLAIN: {
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "2.00"
    },
    "table": {
      "table_name": "tb_book",
      "access_type": "ALL",
      "rows_examined_per_scan": 5,
      "rows_produced_per_join": 5,
      "filtered": "100.00",
      "cost_info": {
        "read_cost": "1.00",
        "eval_cost": "1.00",
        "prefix_cost": "2.00",
        "data_read_per_join": "1K"
      },
      "used_columns": [
        "book_id",
        "book_name",
        "author"
      ]
    }
  }
}
1 row in set, 1 warning (0.00 sec)
```
下面重点看一下比较重要的几个字段。

### id列

`id`是`select`的唯一标识，有几个`select`就有几个id，并且id的顺序是按`select`出现的顺序增长的，id值越大执行优先级越高，id相同则从上往下执行，id为NULL最后执行。

为了验证上面的结论，临时关闭mysql`5.7`对子查询(`sub queries`)产生的衍生表(`derived tables`)的合并优化
```sh
set session optimizer_switch='derived_merge=off';
```
详情见：
`https://dev.mysql.com/doc/refman/5.7/en/switchable-optimizations.html`

`https://dev.mysql.com/doc/refman/5.7/en/derived-table-optimization.html`

```sh
mysql> set session optimizer_switch='derived_merge=off';
Query OK, 0 rows affected (0.00 sec)

mysql> select (select count(1) from tb_book) as book_count, (select count(1) from tb_hero) as hero_count from (select * from tb_book_hero) as book_hero;
+------------+------------+
| book_count | hero_count |
+------------+------------+
|          5 |          6 |
|          5 |          6 |
|          5 |          6 |
|          5 |          6 |
|          5 |          6 |
|          5 |          6 |
+------------+------------+
6 rows in set (0.00 sec)

mysql> explain select (select count(1) from tb_book) as book_count, (select count(1) from tb_hero) as hero_count from (select * from tb_book_hero) as book_hero;
+----+-------------+--------------+------------+-------+---------------+-----------------------+---------+------+------+----------+-------------+
| id | select_type | table        | partitions | type  | possible_keys | key                   | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------------+------------+-------+---------------+-----------------------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | <derived4>   | NULL       | ALL   | NULL          | NULL                  | NULL    | NULL |    6 |   100.00 | NULL        |
|  4 | DERIVED     | tb_book_hero | NULL       | ALL   | NULL          | NULL                  | NULL    | NULL |    6 |   100.00 | NULL        |
|  3 | SUBQUERY    | tb_hero      | NULL       | index | NULL          | idx_book_id_hero_name | 136     | NULL |    6 |   100.00 | Using index |
|  2 | SUBQUERY    | tb_book      | NULL       | index | NULL          | uk_book_name          | 259     | NULL |    5 |   100.00 | Using index |
+----+-------------+--------------+------------+-------+---------------+-----------------------+---------+------+------+----------+-------------+
4 rows in set, 1 warning (0.00 sec)

mysql> set session optimizer_switch='derived_merge=on';
Query OK, 0 rows affected (0.00 sec)
```
可见，查询语句中有4个select，先执行的是`select * from tb_book_hero`，然后执行`select count(1) from tb_hero`，再执行`select count(1) from tb_book`，最后执行`select book_count, hero_count from book_hero`

### select_type列

`select_type`表示的是查询类型，常见的包括`SIMPLE`、`PRIMARY`、`SUBQUERY`、`DERIVED`、`UNION`

(1) SIMPLE：简单查询(不包含子查询和UNION查询)

```sh
mysql> explain select * from tb_book where book_id = 1;
+----+-------------+---------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+---------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | tb_book | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+---------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

(2) PRIMARY：复杂查询中最外层的查询  
(3) SUBQUERY：包含在select中的子查询(不在from子句中)  
(4) DERIVED：包含在from子句中的子查询，MySQL会将结果存放在一个临时表中，也称为派生表(`derived tables`)

这3种select_type见下面的例子
```sh
mysql> set session optimizer_switch='derived_merge=off';
Query OK, 0 rows affected (0.00 sec)

mysql> explain select (select count(1) from tb_book) as book_count from (select * from tb_book_hero) as book_hero;
+----+-------------+--------------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
| id | select_type | table        | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | <derived3>   | NULL       | ALL   | NULL          | NULL         | NULL    | NULL |    6 |   100.00 | NULL        |
|  3 | DERIVED     | tb_book_hero | NULL       | ALL   | NULL          | NULL         | NULL    | NULL |    6 |   100.00 | NULL        |
|  2 | SUBQUERY    | tb_book      | NULL       | index | NULL          | uk_book_name | 259     | NULL |    5 |   100.00 | Using index |
+----+-------------+--------------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)

mysql> set session optimizer_switch='derived_merge=on';
Query OK, 0 rows affected (0.00 sec)
```

(5) UNION：在UNION中的第二个和随后的select

```sh
mysql> select * from tb_book where book_id = 1 union all select * from tb_book where book_name = '笑傲江湖';
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情剑客无情剑        | 古龙   |
|       2 | 笑傲江湖              | 金庸   |
+---------+-----------------------+--------+
2 rows in set (0.00 sec)

mysql> explain select * from tb_book where book_id = 1 union all select * from tb_book where book_name = '笑傲江湖';
+----+-------------+---------+------------+-------+---------------+--------------+---------+-------+------+----------+-------+
| id | select_type | table   | partitions | type  | possible_keys | key          | key_len | ref   | rows | filtered | Extra |
+----+-------------+---------+------------+-------+---------------+--------------+---------+-------+------+----------+-------+
|  1 | PRIMARY     | tb_book | NULL       | const | PRIMARY       | PRIMARY      | 4       | const |    1 |   100.00 | NULL  |
|  2 | UNION       | tb_book | NULL       | const | uk_book_name  | uk_book_name | 259     | const |    1 |   100.00 | NULL  |
+----+-------------+---------+------------+-------+---------------+--------------+---------+-------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)
```

### table列

`table`表示查询涉及的表或衍生表。

常见table列是`<derivenN>`格式，表示当前查询依赖`id=N`的查询，需先执行`id=N`的查询。上面含`select_type`为`DERIVED`的查询就是这种情况，这里不再重复举例。

### type列

相关文档：  
`https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-join-types`

type列是判断查询是否高效的重要依据，我们可以通过type字段的值，判断此次查询是`全表扫描`还是`索引扫描`等，进而进一步优化查询。

一般来说表示查询性能最优到最差依次为：`NULL > system > const > eq_ref > ref > range > index > ALL`

前面的几种类型都是利用到了索引来查询数据, 因此可以过滤部分或大部分数据, 查询效率自然就比较高了。
而后面的`index`类型的查询虽然不是全表扫描, 但是它扫描了所有的索引, 因此比`ALL`类型稍快。
所以，应当尽可能地保证查询达到`range`级别，最好达到`ref`。


(0) NULL: 不用访问表或者索引，直接就能得到结果，如：在索引列中选取最大值，执行时不需要再访问表
```sh
mysql> explain select max(book_id) from tb_book;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
1 row in set, 1 warning (0.00 sec)
```

(1) system：The table has only one row. This is a special case of the `const` join type.

当查询的表只有一行的情况下，`system`是`const`类型的特例，

(2) const：It is used when you compare all parts of a `PRIMARY KEY` or `UNIQUE index` to `constant values`.

针对`主键`或`唯一索引`的等值查询扫描, 最多只返回一行数据。`const`查询速度非常快, 因为它仅仅读取一次即可。

关于type列为`system`、`const`的情况，见下面的示例：
```sh
mysql> set session optimizer_switch='derived_merge=off';
Query OK, 0 rows affected (0.00 sec)

mysql> explain select * from (select * from tb_book where book_id = 5) as book;
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table      | partitions | type   | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
|  1 | PRIMARY     | <derived2> | NULL       | system | NULL          | NULL    | NULL    | NULL  |    1 |   100.00 | NULL  |
|  2 | DERIVED     | tb_book    | NULL       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)

mysql> set session optimizer_switch='derived_merge=on';
Query OK, 0 rows affected (0.00 sec)
```

(3) eq_ref：It is used when all parts of an index are used by the join and the index is a `PRIMARY KEY` or `UNIQUE NOT NULL index`.

此类型通常出现在多表的join查询，表示对于前表的每一个结果，都只能匹配到后表的一行结果，并且查询的比较操作通常是`=`，查询效率较高。

```sh
mysql> select tb_hero.*, tb_book_hero.user_comment from tb_book_hero, tb_hero where tb_book_hero.book_id = 2 and tb_book_hero.hero_id = tb_hero.hero_id;
+---------+-----------+--------------+---------+--------------------------------------------------------------------------------------+
| hero_id | hero_name | skill        | book_id | user_comment                                                                         |
+---------+-----------+--------------+---------+--------------------------------------------------------------------------------------+
|       2 | 令狐冲    | 独孤九剑     |       2 | 令狐少侠留步!                                                                        |
|       6 | 任我行    | 吸星大法     |       2 | 有人就有恩怨，有恩怨就有江湖，人心即是江湖，你如何退出！                             |
+---------+-----------+--------------+---------+--------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> explain select tb_hero.*, tb_book_hero.user_comment from tb_book_hero, tb_hero where tb_book_hero.book_id = 2 and tb_book_hero.hero_id = tb_hero.hero_id;
+----+-------------+--------------+------------+--------+---------------+---------+---------+-----------------------------------+------+----------+-------+
| id | select_type | table        | partitions | type   | possible_keys | key     | key_len | ref                               | rows | filtered | Extra |
+----+-------------+--------------+------------+--------+---------------+---------+---------+-----------------------------------+------+----------+-------+
|  1 | SIMPLE      | tb_book_hero | NULL       | ref    | PRIMARY       | PRIMARY | 4       | const                             |    2 |   100.00 | NULL  |
|  1 | SIMPLE      | tb_hero      | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | explain_test.tb_book_hero.hero_id |    1 |   100.00 | NULL  |
+----+-------------+--------------+------------+--------+---------------+---------+---------+-----------------------------------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)

mysql> select tb_hero.*, tb_book_hero.user_comment from tb_book_hero join tb_hero on tb_book_hero.book_id = 2 and tb_book_hero.hero_id = tb_hero.hero_id;
+---------+-----------+--------------+---------+--------------------------------------------------------------------------------------+
| hero_id | hero_name | skill        | book_id | user_comment                                                                         |
+---------+-----------+--------------+---------+--------------------------------------------------------------------------------------+
|       2 | 令狐冲    | 独孤九剑     |       2 | 令狐少侠留步!                                                                        |
|       6 | 任我行    | 吸星大法     |       2 | 有人就有恩怨，有恩怨就有江湖，人心即是江湖，你如何退出！                             |
+---------+-----------+--------------+---------+--------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> explain select tb_hero.*, tb_book_hero.user_comment from tb_book_hero join tb_hero on tb_book_hero.book_id = 2 and tb_book_hero.hero_id = tb_hero.hero_id;
+----+-------------+--------------+------------+--------+---------------+---------+---------+-----------------------------------+------+----------+-------+
| id | select_type | table        | partitions | type   | possible_keys | key     | key_len | ref                               | rows | filtered | Extra |
+----+-------------+--------------+------------+--------+---------------+---------+---------+-----------------------------------+------+----------+-------+
|  1 | SIMPLE      | tb_book_hero | NULL       | ref    | PRIMARY       | PRIMARY | 4       | const                             |    2 |   100.00 | NULL  |
|  1 | SIMPLE      | tb_hero      | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | explain_test.tb_book_hero.hero_id |    1 |   100.00 | NULL  |
+----+-------------+--------------+------------+--------+---------------+---------+---------+-----------------------------------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)
```

(4) ref: It is used if the join uses only a leftmost prefix of the key or if the key is not a PRIMARY KEY or UNIQUE index (in other words, if the join cannot select a single row based on the key value).

相比`eq_ref`，不使用唯一索引，而是使用普通索引或者唯一性索引的最左前缀，可能会找到多个符合条件的行。

- 简单的`select`查询，`author`列上建有普通索引（非唯一索引）
```sh
mysql> explain select * from tb_book where author = '古龙';
+----+-------------+---------+------------+------+---------------+------------+---------+-------+------+----------+-------+
| id | select_type | table   | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra |
+----+-------------+---------+------------+------+---------------+------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | tb_book | NULL       | ref  | idx_author    | idx_author | 131     | const |    2 |   100.00 | NULL  |
+----+-------------+---------+------------+------+---------------+------------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

- 关联表查询，`tb_book_hero`表使用了联合主键`PRIMARY KEY (book_id, hero_id)`，这里使用到了左边前缀`book_id`进行过滤。
```sh
mysql> explain select * from tb_book_hero where book_id = 3;
+----+-------------+--------------+------------+------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table        | partitions | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+--------------+------------+------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | tb_book_hero | NULL       | ref  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+--------------+------------+------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

(5) range: It can be used when a key column is compared to a constant using any of the =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, LIKE, or IN() operators  
扫描部分索引(范围扫描)，对索引的扫描开始于某一点，返回匹配值域的行，常见于between、<、>、in等查询
```sh
mysql> explain select * from tb_book where book_id > 3;
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | tb_book | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL |    2 |   100.00 | Using where |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

(6) index：the index tree is scanned, MySQL can use this type when the query uses only columns that are part of a single index.  
表示全索引扫描(full index scan), 和ALL类型类似, 只不过ALL类型是全表扫描, 而index类型则仅仅扫描所有的索引, 而不扫描数据.
```sh
mysql> explain select book_name from tb_book;
+----+-------------+---------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | tb_book | NULL       | index | NULL          | uk_book_name | 259     | NULL |    5 |   100.00 | Using index |
+----+-------------+---------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
上面的例子中, 我们查询的`book_name`字段上恰好有索引, 因此我们直接从索引中获取数据就可以满足查询的需求了, 而不需要查询表中的数据。因此这样的情况下, type的值是index, 并且Extra的值大多是`Using index`。

(7) ALL: A full table scan is done  
表示全表扫描, 这个类型的查询是性能最差的查询之一。通常来说, 我们的查询不应该出现ALL类型的查询, 因为这样的查询在数据量大的情况下, 严重降低数据库的性能。如果一个查询是ALL类型查询, 那么大多可以对相应的字段添加索引来避免。
```sh
mysql> explain select * from tb_hero where hero_name = '令狐冲';
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | tb_hero | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    6 |    16.67 | Using where |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

### possible_keys列
表示MySQL在查询时, 能够使用到的索引。注意, 即使有些索引在possible_keys中出现, 但是并不表示此索引会真正地被MySQL使用到。MySQL在查询时具体使用了哪些索引, 由key字段决定。

### key列
这一列显示mysql实际采用哪个索引来优化对该表的访问。如果没有使用索引，则该列是NULL。

### key_len列
表示查询优化器使用了索引的字节数，这个字段可以评估联合索引是否完全被使用, 或只有最左部分字段被使用到。
举例来说，`tb_hero`表的联合索引`idx_book_id_hero_name`由`book_id`和`hero_name`两个列组成，int类型占4字节，另外如果字段允许为NULL，需要1字节记录是否为NULL，通过结果中的key_len=5(`tb_hero`.`book_id`允许为NULL)可推断出查询使用了第一个列`book_id`列来执行索引查找；再拿`tb_book_hero`表联合主键`PRIMARY KEY (book_id, hero_id)`举例，通过key_len=4(`tb_book_hero`.`book_id`不允许为NULL)可推断出查询使用了第一个列`book_id`列来执行索引查找
```sh
mysql> explain select * from tb_hero where book_id = 2;
+----+-------------+---------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-------+
| id | select_type | table   | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra |
+----+-------------+---------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | tb_hero | NULL       | ref  | idx_book_id_hero_name | idx_book_id_hero_name | 5       | const |    2 |   100.00 | NULL  |
+----+-------------+---------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select * from tb_book_hero where book_id = 2;
+----+-------------+--------------+------------+------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table        | partitions | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+--------------+------------+------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | tb_book_hero | NULL       | ref  | PRIMARY       | PRIMARY | 4       | const |    2 |   100.00 | NULL  |
+----+-------------+--------------+------------+------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.01 sec)
```

`key_len`的计算规则如下:

- 字符串:
    - char(n): n字节长度
    - varchar(n): 如果是`utf8`编码, 则是`3n + 2`字节; 如果是`utf8mb4`编码, 则是`4n + 2`字节.

- 数值类型:
    - TINYINT: 1字节
    - SMALLINT: 2字节
    - MEDIUMINT: 3字节
    - INT: 4字节
    - BIGINT: 8字节

- 时间类型
    - DATE: 3字节
    - TIMESTAMP: 4字节
    - DATETIME: 8字节

- 字段属性:
    - NULL属性占用一个字节
    - 如果一个字段是NOT NULL的, 则没有此属性

再看下面的计算:  
`4 [book_id是int类型] + 1 [book_id允许为NULL] + (4 * 32 + 2) [hero_name是varchar32,且用的是utf8mb4编码] + 1 [hero_name允许为NULL] =  136`
```sh
mysql> explain select * from tb_hero where book_id = 2 and hero_name = '令狐冲';
+----+-------------+---------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-------+
| id | select_type | table   | partitions | type | possible_keys         | key                   | key_len | ref         | rows | filtered | Extra |
+----+-------------+---------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-------+
|  1 | SIMPLE      | tb_hero | NULL       | ref  | idx_book_id_hero_name | idx_book_id_hero_name | 136     | const,const |    1 |   100.00 | NULL  |
+----+-------------+---------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

### ref列

> The ref column shows which columns or constants are compared to the index named in the key column to select rows from the table.
显示的是哪个字段或常数与key一起被使用

### rows列

MySQL查询优化器根据统计信息, 估算SQL要查找到结果集需要扫描读取的数据行数，注意这个不是结果集里的行数。这个值非常直观显示SQL的效率好坏, 原则上rows越少越好。

### Extra列

这一列展示的是额外信息。常见的重要值如下:

(1) Using index

表示查询在索引树中就可查到所需数据, 不用扫描表数据文件

```sh
mysql> explain select hero_id from tb_book_hero where book_id = 2;
+----+-------------+--------------+------------+------+---------------+---------+---------+-------+------+----------+-------------+
| id | select_type | table        | partitions | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra       |
+----+-------------+--------------+------------+------+---------------+---------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | tb_book_hero | NULL       | ref  | PRIMARY       | PRIMARY | 4       | const |    2 |   100.00 | Using index |
+----+-------------+--------------+------------+------+---------------+---------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.01 sec)

mysql> explain select book_id  from tb_book where author = '金庸';
+----+-------------+---------+------------+------+---------------+------------+---------+-------+------+----------+-------------+
| id | select_type | table   | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra       |
+----+-------------+---------+------------+------+---------------+------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | tb_book | NULL       | ref  | idx_author    | idx_author | 131     | const |    3 |   100.00 | Using index |
+----+-------------+---------+------------+------+---------------+------------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

(2) Using where
查询的列没有全部被索引覆盖
```sh
mysql> explain select book_id, book_name from tb_book where author = '金庸';
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | tb_book | NULL       | ALL  | idx_author    | NULL | NULL    | NULL |    5 |    60.00 | Using where |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

(3) Using temporary

查询有使用临时表，一般出现于排序、分组、多表join、distinct查询等等。

举例子如下：`tb_book`表对`book_name`字段建立了唯一性索引，这时候distinct查询Extra列为`Using index`; `tb_hero`表的`skill`字段上没有任何索引，这时候distinct查询Extra列为`Using temporary`

```sh
mysql> select distinct book_name from tb_book;
+-----------------------+
| book_name             |
+-----------------------+
| 倚天屠龙记            |
| 多情剑客无情剑        |
| 射雕英雄传            |
| 笑傲江湖              |
| 绝代双骄              |
+-----------------------+
5 rows in set (0.00 sec)

mysql> explain select distinct book_name from tb_book;
+----+-------------+---------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | tb_book | NULL       | index | uk_book_name  | uk_book_name | 259     | NULL |    5 |   100.00 | Using index |
+----+-------------+---------+------------+-------+---------------+--------------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> select distinct skill from tb_hero;
+-----------------+
| skill           |
+-----------------+
| 小李飞刀        |
| 独孤九剑        |
| 九阳神功        |
| 降龙十八掌      |
| 移花接玉        |
| 吸星大法        |
+-----------------+
6 rows in set (0.00 sec)

mysql> explain select distinct skill from tb_hero;
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | SIMPLE      | tb_hero | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    6 |   100.00 | Using temporary |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-----------------+
1 row in set, 1 warning (0.00 sec)
```

(4) Using filesort

表示MySQL不能通过索引顺序达到排序效果，需额外的排序操作，数据较小时在内存排序，否则需要在磁盘完成排序。这种情况下一般也是要考虑使用索引来优化的。

```sh
mysql> explain select book_id, hero_name from tb_hero order by hero_name;
+----+-------------+---------+------------+-------+---------------+-----------------------+---------+------+------+----------+-----------------------------+
| id | select_type | table   | partitions | type  | possible_keys | key                   | key_len | ref  | rows | filtered | Extra                       |
+----+-------------+---------+------------+-------+---------------+-----------------------+---------+------+------+----------+-----------------------------+
|  1 | SIMPLE      | tb_hero | NULL       | index | NULL          | idx_book_id_hero_name | 136     | NULL |    6 |   100.00 | Using index; Using filesort |
+----+-------------+---------+------------+-------+---------------+-----------------------+---------+------+------+----------+-----------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select book_id, hero_name from tb_hero order by book_id, hero_name;
+----+-------------+---------+------------+-------+---------------+-----------------------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key                   | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+-----------------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | tb_hero | NULL       | index | NULL          | idx_book_id_hero_name | 136     | NULL |    6 |   100.00 | Using index |
+----+-------------+---------+------------+-------+---------------+-----------------------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
`tb_hero`表上有联合索引`INDEX idx_book_id_hero_name(book_id, hero_name) USING BTREE`  
但是`order by hero_name`, 不能使用索引进行优化(下一篇博客会介绍联合索引的结构), 进而会产生`Using filesort`  
如果将排序依据改为`order by book_id, hero_name`, 就不会出现`Using filesort`了。

(5) Select tables optimized away
比如下面的例子：
```sh
mysql> explain select min(book_id), max(book_id) from tb_book;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
1 row in set, 1 warning (0.00 sec)
```