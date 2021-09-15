# 一文读懂MySQL的事务隔离级别及MVCC机制

回顾前文:  
[一文学会MySQL的explain工具](https://www.cnblogs.com/itwild/p/13424113.html)

[一文读懂MySQL的索引结构及查询优化](https://www.cnblogs.com/itwild/p/13703259.html)

(同时再次强调，这几篇关于MySQL的探究都是基于`5.7`版本，相关总结与结论`不一定适用`于其他版本)

就软件开发而言，既要保证数据读写的`效率`，还要保证`并发读写`数据的`可靠性`、`正确性`。因此，除了要对MySQL的索引结构及查询优化有所了解外，还需要对MySQL的事务隔离级别及MVCC机制有所认知。

MySQL官方文档中的词汇表(`https://dev.mysql.com/doc/refman/5.7/en/glossary.html`)有助于我们对相关概念、理论的理解。下文中我会从概念表中摘录部分原文描述，以加深对原理机制的理解。

## 事务隔离级别

### 事务是什么

> Transactions are atomic units of work that can be committed or rolled back. When a transaction makes multiple changes to the database, either all the changes succeed when the transaction is committed, or all the changes are undone when the transaction is rolled back.

事务是由一组SQL语句组成的原子操作单元，其对数据的变更，要么全都执行成功(`Committed`)，要么全都不执行(`Rollback`)。

![事务的示意图](https://img2020.cnblogs.com/blog/1546632/202010/1546632-20201003090232837-258120887.png)

> Database transactions, as implemented by InnoDB, have properties that are collectively known by the acronym ACID, for atomicity, consistency, isolation, and durability.

`InnoDB`实现的数据库事务具有常说的`ACID`属性，即原子性(`atomicity`)，一致性(`consistency`)、隔离性(`isolation`)和持久性(`durability`)。

- `原子性`：事务被视为不可分割的最小单元，所有操作要么全部执行成功，要么失败回滚(即还原到事务开始前的状态，就像这个事务从来没有执行过一样)
- `一致性`：在成功提交或失败回滚之后以及正在进行的事务期间，数据库始终保持一致的状态。如果正在多个表之间更新相关数据，那么查询将看到所有旧值或所有新值，而不会一部分是新值，一部分是旧值
- `隔离性`：事务处理过程中的中间状态应该对外部不可见，换句话说，事务在进行过程中是隔离的，事务之间不能互相干扰，不能访问到彼此未提交的数据。这种隔离可通过锁机制实现。有经验的用户可以根据实际的业务场景，通过调整事务隔离级别，以提高并发能力
- `持久性`：一旦事务提交，其所做的修改将会永远保存到数据库中。即使系统发生故障，事务执行的结果也不能丢失

> In InnoDB, all user activity occurs inside a transaction. If autocommit mode is enabled, each SQL statement forms a single transaction on its own. By default, MySQL starts the session for each new connection with autocommit enabled, so MySQL does a commit after each SQL statement if that statement did not return an error. If a statement returns an error, the commit or rollback behavior depends on the error

MySQL默认采用自动提交(`autocommit`)模式。也就是说，如果不显式使用`START TRANSACTION`或`BEGIN`语句来开启一个事务，那么每个SQL语句都会被当做一个事务自动提交。

> A session that has autocommit enabled can perform a multiple-statement transaction by starting it with an explicit START TRANSACTION or BEGIN statement and ending it with a COMMIT or ROLLBACK statement.

多个SQL语句开启一个事务也很简单，以`START TRANSACTION`或者`BEGIN`语句开头，以`COMMIT`或`ROLLBACK`语句结尾。

> If autocommit mode is disabled within a session with SET autocommit = 0, the session always has a transaction open. A COMMIT or ROLLBACK statement ends the current transaction and a new one starts.

使用`SET autocommit = 0`可手动关闭当前`session`自动提交模式。

### 并发事务的问题

#### 引出事务隔离级别

相关文档：`https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html`

> Isolation is the I in the acronym ACID; the isolation level is the setting that fine-tunes the balance between performance and reliability, consistency, and reproducibility of results when multiple transactions are making changes and performing queries at the same time.

也就是说当多个并发请求访问MySQL，其中有对数据的增删改请求时，考虑到并发性，又为了避免`脏读`、`不可重复读`、`幻读`等问题，就需要对事务之间的读写进行隔离，至于隔离到啥程度需要看具体的业务场景，这时就要引出事务的隔离级别了。

> InnoDB offers all four transaction isolation levels described by the SQL:1992 standard: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, and SERIALIZABLE. The default isolation level for InnoDB is REPEATABLE READ.

`InnoDB`存储引擎实现了SQL标准中描述的4个事务隔离级别：读未提交(`READ UNCOMMITTED`)、读已提交(`READ COMMITTED`)、可重复读(`REPEATABLE READ`)、可串行化(`SERIALIZABLE`)。`InnoDB`默认隔离级别是可重复读(`REPEATABLE READ`)。

#### 设置事务隔离级别

既然可以调整隔离级别，那么如何设置事务隔离级别呢？详情见官方文档：`https://dev.mysql.com/doc/refman/5.7/en/set-transaction.html`

MySQL`5.7.18`版本演示如下：
```sh
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.18    |
+-----------+
1 row in set (0.00 sec)

mysql> set global transaction isolation level REPEATABLE READ;
Query OK, 0 rows affected (0.00 sec)

mysql> set session transaction isolation level READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> select @@global.tx_isolation, @@session.tx_isolation, @@tx_isolation;
+-----------------------+------------------------+----------------+
| @@global.tx_isolation | @@session.tx_isolation | @@tx_isolation |
+-----------------------+------------------------+----------------+
| REPEATABLE-READ       | READ-COMMITTED         | READ-COMMITTED |
+-----------------------+------------------------+----------------+
1 row in set (0.00 sec)
```

MySQL`8.0.21`版本演示如下：
```sh
mysql> select version();
+-----------+
| version() |
+-----------+
| 8.0.21    |
+-----------+
1 row in set (0.01 sec)

mysql> set global transaction isolation level REPEATABLE READ;
Query OK, 0 rows affected (0.00 sec)

mysql> set session transaction isolation level READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> select @@global.transaction_isolation, @@session.transaction_isolation, @@transaction_isolation;
+--------------------------------+---------------------------------+-------------------------+
| @@global.transaction_isolation | @@session.transaction_isolation | @@transaction_isolation |
+--------------------------------+---------------------------------+-------------------------+
| REPEATABLE-READ                | READ-COMMITTED                  | READ-COMMITTED          |
+--------------------------------+---------------------------------+-------------------------+
1 row in set (0.00 sec)
```

**注意**：

> transaction_isolation was added in MySQL 5.7.20 as a synonym for tx_isolation, which is now deprecated and is removed in MySQL 8.0. Applications should be adjusted to use transaction_isolation in preference to tx_isolation.

> Prior to MySQL 5.7.20, use tx_isolation and tx_read_only rather than transaction_isolation and transaction_read_only.

如果使用系统变量(`system variables`)来查看或者设置事务隔离级别，需要注意MySQL的版本。在MySQL`5.7.20`之前，应使用`tx_isolation`；在MySQL`5.7.20`之后，应使用`transaction_isolation`。

> You can set transaction characteristics globally, for the current session, or for the next transaction only.

事务的隔离级别范围(`Transaction Characteristic Scope`)可以精确到全局(`global`)、当前会话(`session`)、甚至是仅针对下一个事务生效(`the next transaction only`)。

- 含`global`关键词时，事务隔离级别的设置应用于所有后续`session`，已存在的`session`不受影响
- 含`session`关键词时，事务隔离级别的设置应用于在当前`session`中执行的所有后续事务，不会影响当前正在进行的事务
- 不含`global`以及`session`关键词时，事务隔离级别的设置仅应用于在当前`session`中执行的下一个事务

#### 数据准备

为了演示`脏读`、`不可重复读`、`幻读`等问题，准备了一些初始化数据如下：

```sql
-- ----------------------------
--  create database
-- ----------------------------
create database `transaction_test` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- switch database
use `transaction_test`;

-- ----------------------------
--  table structure for `tb_book`
-- ----------------------------
CREATE TABLE `tb_book` (
  `book_id` int(11) NOT NULL,
  `book_name` varchar(64) DEFAULT NULL,
  `author` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`book_id`),
  UNIQUE KEY `uk_book_name` (`book_name`) USING BTREE
) ENGINE = InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

BEGIN;
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (1, '多情剑客无情剑', '古龙');
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (2, '笑傲江湖', '金庸');
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (3, '倚天屠龙记', '金庸');
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (4, '射雕英雄传', '金庸');
INSERT INTO `tb_book`(`book_id`, `book_name`, `author`) VALUES (5, '绝代双骄', '古龙');
COMMIT;
```

#### 脏读(read uncommitted)

**事务A读到了事务B已经修改但尚未提交的数据**

操作：
1. `session A`事务隔离级别设置为`read uncommitted`并开启事务，首次查询`book_id`为1的记录；
2. 然后`session B`开启事务，并修改`book_id`为1的记录，不提交事务，在`session A`中再次查询`book_id`为1的记录；
3. 最后让`session B`中的事务回滚，再在`session A`中查询`book_id`为1的记录。

session A：

```sql
mysql> set session transaction isolation level read uncommitted;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from tb_book where book_id = 1;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情剑客无情剑        | 古龙   |
+---------+-----------------------+--------+
1 row in set (0.00 sec)

mysql> select * from tb_book where book_id = 1;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情刀客无情刀        | 古龙   |
+---------+-----------------------+--------+
1 row in set (0.00 sec)

mysql> select * from tb_book where book_id = 1;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情剑客无情剑        | 古龙   |
+---------+-----------------------+--------+
1 row in set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

session B：

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update tb_book set book_name = '多情刀客无情刀' where book_id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)
```

结果：`事务A`读到了`事务B`还没提交的中间状态，即产生了`脏读`。

#### 不可重复读(read committed)

**事务A读到了事务B已经提交的修改数据**

操作：
1. `session A`事务隔离级别设置为`read committed`并开启事务，首次查询`book_id`为1的记录；
2. 然后`session B`开启事务，并修改`book_id`为1的记录，不提交事务，在`session A`中再次查询`book_id`为1的记录；
3. 最后提交`session B`中的事务，再在`session A`中查看`book_id`为1的记录。

session A：

```sql
mysql> set session transaction isolation level read committed;
Query OK, 0 rows affected (0.01 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from tb_book where book_id = 1;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情剑客无情剑        | 古龙   |
+---------+-----------------------+--------+
1 row in set (0.00 sec)

mysql> select * from tb_book where book_id = 1;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情剑客无情剑        | 古龙   |
+---------+-----------------------+--------+
1 row in set (0.00 sec)

mysql> select * from tb_book where book_id = 1;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情刀客无情刀        | 古龙   |
+---------+-----------------------+--------+
1 row in set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

session B：

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update tb_book set book_name = '多情刀客无情刀' where book_id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```
结果：`事务B`没有提交事务时，`事务A`不会读到`事务B`修改的中间状态，即`read committed`解决了上面所说的`脏读`问题，但是当`事务B`中的事务提交后，`事务A`读到了修改后的记录，而对于`事务A`来说，仅仅读了两次，却读到了两个不同的结果，违背了事务之间的隔离性，所以说该事务隔离级别下产生了`不可重复读`的问题。

#### 幻读(repeatable read)

**事务A读到了事务B提交的新增数据**

操作：
1. `session A`事务隔离级别设置为`repeatable read`并开启事务，并查询`book`列表
2. `session B`开启事务，先修改`book_id`为5的记录，再插入一条新的数据，提交事务，在`session A`中再次查询`book`列表
3. 在`session A`中更新`session B`中新插入的那条数据，再查询`book`列表

session A：

```sql
mysql> set session transaction isolation level repeatable read;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from tb_book;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情刀客无情刀        | 古龙   |
|       2 | 笑傲江湖              | 金庸   |
|       3 | 倚天屠龙记            | 金庸   |
|       4 | 射雕英雄传            | 金庸   |
|       5 | 绝代双骄              | 古龙   |
+---------+-----------------------+--------+
5 rows in set (0.00 sec)

mysql> select * from tb_book;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情刀客无情刀        | 古龙   |
|       2 | 笑傲江湖              | 金庸   |
|       3 | 倚天屠龙记            | 金庸   |
|       4 | 射雕英雄传            | 金庸   |
|       5 | 绝代双骄              | 古龙   |
+---------+-----------------------+--------+
5 rows in set (0.00 sec)

mysql> update tb_book set book_name = '圆月弯剑' where book_id = 6;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from tb_book;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情刀客无情刀        | 古龙   |
|       2 | 笑傲江湖              | 金庸   |
|       3 | 倚天屠龙记            | 金庸   |
|       4 | 射雕英雄传            | 金庸   |
|       5 | 绝代双骄              | 古龙   |
|       6 | 圆月弯剑              | 古龙   |
+---------+-----------------------+--------+
6 rows in set (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)
```

session B：

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update tb_book set book_name = '绝代双雄' where book_id = 5;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> insert into tb_book values (6, '圆月弯刀', '古龙');
Query OK, 1 row affected (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from tb_book;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情刀客无情刀        | 古龙   |
|       2 | 笑傲江湖              | 金庸   |
|       3 | 倚天屠龙记            | 金庸   |
|       4 | 射雕英雄传            | 金庸   |
|       5 | 绝代双雄              | 古龙   |
|       6 | 圆月弯刀              | 古龙   |
+---------+-----------------------+--------+
6 rows in set (0.00 sec)
```

结果：`事务B`已提交的修改记录(即`绝代双骄`修改为`绝代双雄`)在`事务A`中是不可见的，说明该事务隔离级别下解决了上面`不可重复读`的问题，但魔幻的是一开始`事务A`中虽然读不到`事务B`中的新增记录，却可以更新这条新增记录，执行更新(`update`)后，在`事务A`中居然可见该新增记录了，这便产生了所谓的`幻读`问题。

**为什么会出现这样莫名其妙的结果？** 别急，后文会慢慢揭开这个神秘的面纱。先看如何解决幻读问题。

#### 串行化(serializable)

`serializable`事务隔离级别可以避免幻读问题，但会极大的降低数据库的并发能力。
> SERIALIZABLE: the isolation level that uses the most conservative locking strategy, to prevent any other transactions from inserting or changing data that was read by this transaction, until it is finished.

操作：
1. `session A`事务隔离级别设置为`serializable`并开启事务，并查询`book`列表，不提交事务；
2. 然后`session B`中分别执行`insert`、`delete`、`update`操作

session A：

```sql
mysql> set session transaction isolation level serializable;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from tb_book;
+---------+-----------------------+--------+
| book_id | book_name             | author |
+---------+-----------------------+--------+
|       1 | 多情刀客无情刀        | 古龙   |
|       2 | 笑傲江湖              | 金庸   |
|       3 | 倚天屠龙记            | 金庸   |
|       4 | 射雕英雄传            | 金庸   |
|       5 | 绝代双雄              | 古龙   |
|       6 | 圆月弯刀              | 古龙   |
+---------+-----------------------+--------+
6 rows in set (0.00 sec)
```

session B：

```sql
mysql> insert into tb_book values (7, '神雕侠侣', '金庸');
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction

mysql> delete from tb_book where book_id = 1;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction

mysql> update tb_book set book_name = '绝代双骄' where book_id = 5;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

结果：只要`session A`中的事务一直不提交，`session B`中尝试更改数据(`insert`、`delete`、`update`)的事务都会被阻塞至超时(`timeout`)。显然，该事务隔离级别下能有效解决上面`幻读`、`不可重复读`、`脏读`等问题。

注意：除非是一些特殊的应用场景需要`serializable`事务隔离级别，否则很少会使用该隔离级别，因为并发性极低。

###  事务隔离级别小结

|事务隔离级别|脏读|不可重复读|幻读|
|----|----|----|----|
|read uncommitted|可能|可能|可能|
|read committed|不可能|可能|可能|
|repeatable read|不可能|不可能|可能|
|serializable|不可能|不可能|不可能|

## MVCC机制

上面在演示`幻读`问题时，出现的结果让人捉摸不透。原来`InnoDB`存储引擎的默认事务隔离级别可重复读(`repeatable read`)，是通过 "行级锁+MVCC"一起实现的。这就不得不去了解MVCC机制了。

相关文档：`https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html`

参考：  
《MySQL中MVCC的正确打开方式（源码佐证）》 `https://blog.csdn.net/Waves___/article/details/105295060`

《InnoDB事务分析-MVCC》`http://www.leviathan.vip/2019/03/20/InnoDB的事务分析-MVCC/`

《Innodb中的事务隔离级别和锁的关系》 `https://tech.meituan.com/2014/08/20/innodb-lock.html`

### MVCC概念

多版本并发控制(`multiversion concurrency control`，即`MVCC`): 指的是一种提高并发的技术。最早期的数据库系统，只有读读之间可以并发，读写、写读、写写都要阻塞。引入多版本之后，只有写写之间相互阻塞，其他三种操作都可以并行，这样大幅度提高了`InnoDB`的并发性能。在内部实现中，`InnoDB`通过`undo log`保存每条数据的多个版本，并且能够提供数据历史版本给用户读，每个事务读到的数据版本可能是不一样的。在同一个事务中，用户只能看到该事务创建快照之前已经提交的修改和该事务本身做的修改。

简单来说，`MVCC`表达的是**维持一个数据的多个版本，使得读写操作没有冲突**这么一个思想。

MVCC在`read committed`和`repeatable read`两个事务隔离级别下工作。

#### 隐藏字段

> Internally, InnoDB adds three fields to each row stored in the database. A 6-byte DB_TRX_ID field indicates the transaction identifier for the last transaction that inserted or updated the row. Also, a deletion is treated internally as an update where a special bit in the row is set to mark it as deleted. Each row also contains a 7-byte DB_ROLL_PTR field called the roll pointer. The roll pointer points to an undo log record written to the rollback segment. If the row was updated, the undo log record contains the information necessary to rebuild the content of the row before it was updated. A 6-byte DB_ROW_ID field contains a row ID that increases monotonically as new rows are inserted. If InnoDB generates a clustered index automatically, the index contains row ID values. Otherwise, the DB_ROW_ID column does not appear in any index.

`InnoDB`存储引擎在每行数据的后面添加了三个隐藏字段，如下图所示：

![表中某行数据示意图](https://img2020.cnblogs.com/blog/1546632/202010/1546632-20201018163556324-1921643603.png)

1. `DB_TRX_ID`(6字节)：表示最近一次对本记录行做修改(`insert`或`update`)的事务ID。至于`delete`操作，`InnoDB`认为是一个`update`操作，不过会更新一个另外的删除位，将行标识为deleted。并非真正删除。

2. `DB_ROLL_PTR`(7字节)：回滚指针，指向当前记录行的`undo log`信息。

3. `DB_ROW_ID`(6字节)：随着新行插入而单调递增的行ID。当表没有主键或唯一非空索引时，`InnoDB`就会使用这个行ID自动产生聚集索引。前文《一文读懂MySQL的索引结构及查询优化》中也有所提及。这个`DB_ROW_ID`跟`MVCC`关系不大。


#### undo log

`undo log`中存储的是老版本数据，当一个事务需要读取记录行时，如果当前记录行不可见，可以顺着`undo log`链表找到满足其可见性条件的记录行版本。

对数据的变更操作主要包括`insert/update/delete`，在`InnoDB`中，`undo log`分为如下两类：
- `insert undo log`: 事务对`insert`新记录时产生的`undo log`, 只在事务回滚时需要, 并且在事务提交后就可以立即丢弃。
- `update undo log`: 事务对记录进行`delete`和`update`操作时产生的`undo log`，不仅在事务回滚时需要，快照读也需要，只有当数据库所使用的快照中不涉及该日志记录，对应的回滚日志才会被`purge`线程删除。

> `Purge`线程：为了实现`InnoDB`的`MVCC`机制，更新或者删除操作都只是设置一下旧记录的`deleted_bit`，并不真正将旧记录删除。为了节省磁盘空间，`InnoDB`有专门的`purge`线程来清理`deleted_bit`为`true`的记录。`purge`线程自己也维护了一个`read view`，如果某个记录的`deleted_bit`为`true`，并且`DB_TRX_ID`相对于`purge`线程的`read view`可见，那么这条记录一定是可以被安全清除的。

不同事务或者相同事务的对同一记录行的修改形成的`undo log`如下图所示：

![undo log的示意图](https://img2020.cnblogs.com/blog/1546632/202010/1546632-20201018214116601-1252297826.png)

可见链首就是最新的记录，链尾就是最早的旧记录。

#### Read View结构

`Read View`(读视图)提供了某一时刻事务系统的快照，主要是用来做`可见性`判断的, 里面保存了"对本事务不可见的其他活跃事务"。

MySQL`5.7`源码中对`Read View`定义如下(详情见`https://github.com/mysql/mysql-server/blob/5.7/storage/innobase/include/read0types.h#L306`)：

```cpp
class ReadView {
	private:
		/** The read should not see any transaction with trx id >= this
		value. In other words, this is the "high water mark". */
		trx_id_t	m_low_limit_id;

		/** The read should see all trx ids which are strictly
		smaller (<) than this value.  In other words, this is the
		low water mark". */
		trx_id_t	m_up_limit_id;

		/** trx id of creating transaction, set to TRX_ID_MAX for free
		views. */
		trx_id_t	m_creator_trx_id;

		/** Set of RW transactions that was active when this snapshot
		was taken */
		ids_t		m_ids;

		/** The view does not need to see the undo logs for transactions
		whose transaction number is strictly smaller (<) than this value:
		they can be removed in purge if not needed by other views */
		trx_id_t	m_low_limit_no;

		/** AC-NL-RO transaction view that has been "closed". */
		bool		m_closed;

		typedef UT_LIST_NODE_T(ReadView) node_t;

		/** List of read views in trx_sys */
		byte		pad1[64 - sizeof(node_t)];
		node_t		m_view_list;
};
```
重点解释下面几个变量(建议仔细看上面的源码注释，以下仅为个人理解，有理解不到位的地方欢迎指出(●´ω｀●))：

(1) `m_ids`: `Read View`创建时其他未提交的活跃事务ID列表。具体说来就是创建`Read View`时，将当前未提交事务ID记录下来，后续即使它们修改了记录行的值，对于当前事务也是不可见的。注意：该事务ID列表不包括当前事务自己和已提交的事务。

(2) `m_low_limit_id`：某行数据的`DB_TRX_ID >= m_low_limit_id`的任何版本对该查询`不可见`。那么这个值是怎么确定的呢？其实就是读的时刻出现过的最大的事务ID+1，即下一个将被分配的事务ID。见`https://github.com/mysql/mysql-server/blob/5.7/storage/innobase/read/read0read.cc#L459`

```cpp
/**
Opens a read view where exactly the transactions serialized before this
point in time are seen in the view.
@param id		Creator transaction id */

void
ReadView::prepare(trx_id_t id)
{
	m_creator_trx_id = id;

	m_low_limit_no = m_low_limit_id = trx_sys->max_trx_id;
}
```
`max_trx_id`见`https://github.com/mysql/mysql-server/blob/5.7/storage/innobase/include/trx0sys.h#L576`中的描述，翻译过来就是“还未分配的最小事务ID”，也就是下一个将被分配的事务ID。（注意，`m_low_limit_id`并不是活跃事务列表中最大的事务ID）
```cpp
struct trx_sys_t {
/*!< The smallest number not yet
					assigned as a transaction id or
					transaction number. This is declared
					volatile because it can be accessed
					without holding any mutex during
					AC-NL-RO view creation. */
	volatile trx_id_t max_trx_id;
}
```

(3) `m_up_limit_id`：某行数据的`DB_TRX_ID < m_up_limit_id`的所有版本对该查询`可见`。同样这个值又是如何确定的呢？`m_up_limit_id`是活跃事务列表`m_ids`中最小的事务ID，如果trx_ids为空，则`m_up_limit_id`为`m_low_limit_id`。代码见`https://github.com/mysql/mysql-server/blob/5.7/storage/innobase/read/read0read.cc#L485`
```cpp
void
ReadView::complete()
{
	/* The first active transaction has the smallest id. */
	m_up_limit_id = !m_ids.empty() ? m_ids.front() : m_low_limit_id;

	ut_ad(m_up_limit_id <= m_low_limit_id);

	m_closed = false;
}
```
这样就有下面的可见性比较算法了。代码见`https://github.com/mysql/mysql-server/blob/5.7/storage/innobase/include/read0types.h#L169`
```cpp
/** Check whether the changes by id are visible.
	@param[in]	id	transaction id to check against the view
	@param[in]	name	table name
	@return whether the view sees the modifications of id. */
bool changes_visible(
	trx_id_t		id,
	const table_name_t&	name) const
	MY_ATTRIBUTE((warn_unused_result))
{
	ut_ad(id > 0);


	/* 假如 trx_id 小于 Read view 限制的最小活跃事务ID m_up_limit_id 或者等于正在创建的事务ID m_creator_trx_id
     * 即满足事务的可见性.
     */
	if (id < m_up_limit_id || id == m_creator_trx_id) {
		return(true);
	}

	/* 检查 trx_id 是否有效. */
	check_trx_id_sanity(id, name);

	if (id >= m_low_limit_id) {
		/* 假如 trx_id 大于等于m_low_limit_id, 即不可见. */
		return(false);

	} else if (m_ids.empty()) {
		/* 假如目前不存在活跃的事务，即可见. */
		return(true);
	}

	const ids_t::value_type*	p = m_ids.data();

	/* 利用二分查找搜索活跃事务列表
	 * 当 trx_id 在 m_up_limit_id 和 m_low_limit_id 之间
   * 如果 id 在 m_ids 数组中, 表明 ReadView 创建时候，事务处于活跃状态，因此记录不可见.
   */
	return (!std::binary_search(p, p + m_ids.size(), id));
}
```

![事务可见性比较算法图示](https://img2020.cnblogs.com/blog/1546632/202010/1546632-20201018232747852-703197062.png)

完整梳理一下整个过程。

在`InnoDB`中，创建一个新事务后，执行第一个`select`语句的时候，`InnoDB`会创建一个快照（`read view`），快照中会保存系统当前不应该被本事务看到的其他活跃事务id列表（即`m_ids`）。当用户在这个事务中要读取某个记录行的时候，`InnoDB`会将该记录行的`DB_TRX_ID`与该`Read View`中的一些变量进行比较，判断是否满足可见性条件。

假设当前事务要读取某一个记录行，该记录行的`DB_TRX_ID`（即最新修改该行的事务ID）为`trx_id`，`Read View`的活跃事务列表`m_ids`中最早的事务ID为`m_up_limit_id`，将在生成这个`Read Vew`时系统出现过的最大的事务ID+1记为`m_low_limit_id`（即还未分配的事务ID）。

具体的比较算法如下:

1. 如果`trx_id < m_up_limit_id`,那么表明“最新修改该行的事务”在“当前事务”创建快照之前就提交了，所以该记录行的值对当前事务是可见的。跳到步骤5。

2. 如果`trx_id >= m_low_limit_id`, 那么表明“最新修改该行的事务”在“当前事务”创建快照之后才修改该行，所以该记录行的值对当前事务不可见。跳到步骤4。

3. 如果`m_up_limit_id <= trx_id < m_low_limit_id`, 表明“最新修改该行的事务”在“当前事务”创建快照的时候可能处于“活动状态”或者“已提交状态”；所以就要对活跃事务列表trx_ids进行查找（源码中是用的二分查找，因为是有序的）

(1) 如果在活跃事务列表`m_ids`中能找到id为`trx_id`的事务，表明①在“当前事务”创建快照前，“该记录行的值”被“id为`trx_id`的事务”修改了，但没有提交；或者②在“当前事务”创建快照后，“该记录行的值”被“id为`trx_id`的事务”修改了（不管有无提交）；这些情况下，这个记录行的值对当前事务都是不可见的，跳到步骤4；

(2) 在活跃事务列表中找不到，则表明“id为`trx_id`的事务”在修改“该记录行的值”后，在“当前事务”创建快照前就已经提交了，所以记录行对当前事务可见，跳到步骤5。

4. 在该记录行的`DB_ROLL_PTR`指针所指向的`undo log`回滚段中，取出最新的的旧事务号`DB_TRX_ID`, 将它赋给`trx_id`，然后跳到步骤1重新开始判断。

5. 将该可见行的值返回。

### read committed与repeatable read的区别

有了上面的知识铺垫后，就可以从本质上区别`read committed`与`repeatable read`这两种事务隔离级别了。

> With REPEATABLE READ isolation level, the snapshot is based on the time when the first read operation is performed. With READ COMMITTED isolation level, the snapshot is reset to the time of each consistent read operation.

在`InnoDB`中的`repeatable read`级别, 事务`begin`之后，执行第一条`select`（读操作）时, 会创建一个快照(`read view`)，将当前系统中活跃的其他事务记录起来；并且在此事务中之后的其他`select`操作都是使用的这个`read view`对象，不会重新创建，直到事务结束。

在`InnoDB`中的`read committed`级别, 事务`begin`之后，执行每条`select`（读操作）语句时，快照会被重置，即会基于当前`select`重新创建一个快照(`read view`)，所以显然该事务隔离级别下会读到其他事务已经提交的修改数据。

那么，现在能解释上面演示`幻读`问题时，出现的诡异结果吗？我的理解是，因为是在`repeatable read`隔离级别下，肯定还是快照读，即第一次`select`后创建的`read view`对象还是不变的，但是在当前事务中`update`一条记录时，会把当前事务ID设置到更新后的记录的隐藏字段`DB_TRX_ID`上，即`id == m_creator_trx_id`显然成立，于是该条记录就可见了，再次执行`select`操作时就多出这条记录了。
```cpp
if (id < m_up_limit_id || id == m_creator_trx_id) {
  return(true);
 }
```

另外，有了这样的基本认知后，如果你在MySQL事务隔离相关问题遇到一些其他看似很神奇的现象，也可以试试能不能解释得通。

## 总结

通过学习MySQL事务隔离级别及`MVCC`原理机制，有助于加深对MySQL的理解与掌握，更为重要的是，如果让你编写一个并发读写的存储程序，`MVCC`的设计与实现或许能给你一些启发。
