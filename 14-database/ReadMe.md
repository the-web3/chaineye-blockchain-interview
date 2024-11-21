# Mysql 和 Postgres 数据库面试题目



## 第一题：MySQL关键字执行顺序？

1. FROM子句：确定数据来源，包括JOIN的表。
2. ON：执行JOIN条件。
3. JOIN：如果有JOIN则进行JOIN操作。
4. WHERE子句：执行过滤条件。
5. GROUP BY子句：根据指定的列分组结果集合。
6. HAVING子句：执行分组过滤条件。
7. SELECT子句：选取指定的列。
8. DISTINCT子句：去除重复数据。
9. ORDER BY子句：最后对结果进行排序。
10. LIMIT子句：限制结果集的数量。


## 第二题：drop、delete与truncate分别在什么场景之下使用？

drop table和truncate table相同：

- 属于DDL
- 不可回滚
- 不可带where
- 删除速度快

区别：

+ drop删除表内容和结构删除，

delete from

- 属于DML
- 可回滚
- 可带where
- 表结构在，表内容要看where执行的情况
- 删除速度慢,需要逐行删除

使用场景：

- 不再需要一张表的时候，用drop
- 想删除部分数据行时候，用delete，并且带上where子句
- 保留表而删除所有数据的时候用truncate



## 第三题：InnoDB与MyISAM的区别是什么？

- InnoDB支持事务，MyISAM不支持；

- InnoDB支持外键，MyISAM不支持；

- 而InnoDB支持行锁，MyISAM只支持表锁；

- InnoDB聚集索引，

- InnoDB中不保存表的行数，如select count(*) from table时，InnoDB需要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(*)语句包含where条件时MyISAM也需要扫描整个表


## 第四题：MySQL索引类型?

+ **B-Tree 索引**

  B-Tree（平衡树）索引是 MySQL 中最常用的索引类型。它适用于全键值、键值范围和键前缀查找。B-Tree 索引适用于等值查询、范围查询和排序操作。

```sql
CREATE INDEX idx_name ON table_name(column_name);
```

+ **哈希索引（Hash Index）**

  哈希索引基于哈希表实现，只适用于等值查询。哈希索引的查找速度非常快，但不适用于范围查询和排序操作。哈希索引通常由存储引擎自动创建和管理，例如 InnoDB 的内存表（Memory 存储引擎）支持哈希索引。

```sql
CREATE TABLE table_name (
id INT PRIMARY KEY,
column_name VARCHAR(255)
) ENGINE=MEMORY;
```

+ **全文索引（Full-Text Index）**

  全文索引用于全文搜索，适用于文本内容的搜索。全文索引可以对文本内容进行分词和匹配，支持自然语言搜索和布尔搜索。

```sql
CREATE FULLTEXT INDEX idx_fulltext ON table_name(column_name);
```

+ **空间索引（Spatial Index）**

  空间索引用于地理空间数据的存储和查询。空间索引支持几何数据的存储和空间操作（例如距离计算、范围查询等）。

```sql
CREATE SPATIAL INDEX idx_spatial ON table_name(column_name);
```

+ **R-Tree 索引**

  R-Tree 索引是一种用于多维数据的空间索引结构，适用于范围查询和空间操作。R-Tree 索引通常用于地理信息系统（GIS）和空间数据库。

```sql
CREATE TABLE table_name (
id INT PRIMARY KEY,
geom GEOMETRY NOT NULL,
SPATIAL INDEX(geom)
) ENGINE=InnoDB;
```

+ **前缀索引**

  只在所选列的值的前面几个字符上创建索引。可以减少索引所占用的磁盘空间，但可能会降低查询效率。

```sql
alter table user add index index2(email(6))
```

+ **组合索引（Composite Index）**

​	组合索引是指在多个列上创建的索引。组合索引可以提高多列查询的性能，但需要注意索引列的顺序。

```sql
CREATE INDEX idx_composite ON table_name(column1, column2);
```

+ **唯一索引（Unique Index）**

  唯一索引确保索引列的值是唯一的。唯一索引可以防止重复值的插入，并且可以提高查询性能。

```sql
CREATE UNIQUE INDEX idx_unique ON table_name(column_name);
```

+ **主键索引（Primary Key Index）**

  主键索引是一种特殊的唯一索引，用于唯一标识表中的每一行记录。主键索引通常是自增的整数列。

```sql
CREATE TABLE table_name (
id INT PRIMARY KEY,
column_name VARCHAR(255)
);
```

+ **外键索引（Foreign Key Index）**

  外键索引用于建立两个表之间的关系。外键索引确保引用表中的外键值存在于被引用表的主键或唯一键中。

```sql
CREATE TABLE parent_table (
id INT PRIMARY KEY
);

CREATE TABLE child_table (
id INT PRIMARY KEY,
parent_id INT,
FOREIGN KEY (parent_id) REFERENCES parent_table(id)
);
```



## 第五题：InnoDB索引为什么使用B+树？



## 第六题：聚簇索引和非聚簇索引的区别？

| 特性             | 聚簇索引                 | 非聚簇索引                   |
| ---------------- | ------------------------ | ---------------------------- |
| 物理存储顺序     | 决定数据行的物理存储顺序 | 与数据行分开存储             |
| 唯一性           | 每个表只能有一个聚簇索引 | 每个表可以有多个非聚簇索引   |
| 查询性能         | 范围查询和排序操作性能好 | 点查询性能好                 |
| 插入和更新性能   | 可能会影响插入和更新性能 | 通常比聚簇索引好             |
| 叶子节点存储内容 | 存储实际的数据行         | 存储索引键和指向数据行的指针 |

聚簇索引决定了表中数据的物理存储顺序，每个表只能有一个聚簇索引，适用于范围查询和排序操作。非聚簇索引与数据行分开存储，每个表可以有多个非聚簇索引，适用于点查询。



## 第七题：MySQL中是否必然包含聚簇索引吗？

当使用InnoDB作为引擎时，如果定义了主键，那么InnoDB会使用主键作为聚簇索引，如果没有定义主键，那么会使用第一个非空的唯一索引（NOT NULL and UNIQUE INDEX）作为聚簇索引，如果既没有主键，也没有合适的非空索引，那么InnoDB会自动生成一个6字节隐藏列 _rowid作为聚簇索引。



## 第八题：覆盖索引和回表查询是什么？





## 第九题：mysql索引失效的情况？

- **使用函数或表达式**

  如果在查询中对索引列使用了函数或表达式，MySQL 可能无法使用该列上的索引。

  ```sql
  SELECT * FROM users WHERE YEAR(birthdate) = 1990;
  ```

- **隐式类型转换**

  如果查询中的列类型与索引列类型不匹配，MySQL 可能会进行隐式类型转换，导致索引失效。

  ```sql
  SELECT * FROM users WHERE age = '30';
  ```

  假设 `age` 列是整数类型，而查询中使用了字符串 `'30'`，MySQL 会进行隐式类型转换，导致索引失效。

- **组合索引未遵循最左匹配原则**



- **`LIKE`查询使用左通配符**

  如果在 `LIKE` 查询中使用了左通配符（例如`_abc`， `%abc`），MySQL 可能无法使用索引。

  ```sql
  SELECT * FROM users WHERE name LIKE '%john%';
  ```

  在这个查询中，`name` 列上的索引可能无法使用，因为查询使用了前置通配符 `%`。

- **使用 `OR` 条件**

  如果查询中使用了 `OR` 条件，并且 `OR` 条件中的列没有全部被索引覆盖，MySQL 可能无法使用索引。

  ```sql
  SELECT * FROM users WHERE age = 30 OR email = 'john@example.com';
  ```

  假设 `age` 列上有索引，而 `email` 列上没有索引，MySQL 可能无法使用索引。

- **使用 `!=`、`<>` 或 `NOT IN`**

  如果查询中使用了 `!=`、`<>` 或 `NOT IN` 条件，MySQL 可能无法使用索引。

  ```sql
  SELECT * FROM users WHERE age != 30;
  ```

  在这个查询中，`age` 列上的索引可能无法使用。

- **使用 `IS NULL` 或 `IS NOT NULL`**

  如果查询中使用了 `IS NULL` 或 `IS NOT NULL` 条件，MySQL 可能无法使用索引。

  ```sql
  SELECT * FROM users WHERE email IS NULL;
  ```

  在这个查询中，`email` 列上的索引可能无法使用。

- **使用 `ORDER BY` 或 `GROUP BY`**

  如果 `ORDER BY` 或 `GROUP BY` 中的列没有被索引覆盖，MySQL 可能无法使用索引。

  ```sql
  SELECT * FROM users ORDER BY last_login;
  ```

  假设 `last_login` 列上没有索引，MySQL 可能无法使用索引进行排序。

- **使用 `SELECT *`**

  使用 `SELECT *` 可能会导致 MySQL 无法有效使用索引，因为返回的数据量过大，可能会导致索引失效。

  

## 第十题： 什么是最左匹配原则?

最左匹配原则（Leftmost Matching Principle）是指在使用组合索引时，查询条件中包含的列必须从组合索引的最左边开始，并且顺序一致，才能利用该索引。



## 第十一题：NOT IN和NOT EXIST的区别？




 
## 第十二题：事务的 ACID 是什么？

**原子性（Atomicity）**： 事务是一个不可分割的工作单元，要么全部成功，要么全部失败。

**实现**: MySQL 通过 `COMMIT` 和 `ROLLBACK` 语句来实现原子性。当事务成功完成时，使用 `COMMIT` 提交事务；如果发生错误，使用 `ROLLBACK` 回滚事务。

**一致性（Consistency）**：假设一个事务将账户 A 的余额转移到账户 B。在事务执行前后，账户 A 和账户 B 的总余额必须保持一致。

**实现**: MySQL 通过触发器、约束（如主键、外键、唯一约束）和存储过程等机制来确保一致性。

**隔离性（Isolation）**：事务之间是相互隔离的，一个事务的执行不会影响其他事务的执行。每个事务都感觉不到其他事务的存在。



**实现**: MySQL 提供了

 **持久性（Durability）**: 事务一旦提交，其结果将永久保存在数据库中，即使系统崩溃也不会丢失。

**实现**: MySQL 通过日志机制（如 `redo log` 和 `undo log`）来确保持久性。`redo log` 记录了事务的操作，`undo log` 记录了事务的回滚信息。在系统崩溃后，MySQL 可以通过日志恢复事务。



**脏读（Dirty Read）**：
一个事务读取到另一个尚未提交事务的修改。

**不可重复读（Non-repeatable Read）**：
在同一个事务内，多次读取同一数据返回的结果有所不同。

**幻读（Phantom Read）**：
一个事务在执行两次相同的查询时，因为另一个并发事务的插入或删除操作，导致两次查询返回的结果集不同。


## 第十三题：MySQL的隔离级别有哪些？

四种隔离级别（`READ UNCOMMITTED`、`READ COMMITTED`、`REPEATABLE READ`、`SERIALIZABLE`）来控制事务的隔离性。默认隔离级别是 `REPEATABLE READ`。

- **READ UNCOMMITTED**: 允许脏读，即一个事务可以读取另一个事务未提交的数据。
- **READ COMMITTED**: 不允许脏读，但允许不可重复读，即一个事务可以读取另一个事务已提交的数据。
- **REPEATABLE READ**: 不允许脏读和不可重复读，但允许幻读，即一个事务可以读取到另一个事务插入的新数据。
- **SERIALIZABLE**: 最高隔离级别，不允许脏读、不可重复读和幻读，所有事务按顺序执行。



## 第十四题：MySQL百万级数据分页要注意什么?



## 第十五题：简述MVCC机制



## 第十六题：间隙锁是什么



## 第十七题：MySQL底层架构体系及每层作用？




## 第十八题：什么是红黑树？



## 第十九题：什么是 B 树？



## 第二十题：什么是 B+ 树？



## 第二十一题：MySQL索引底层实现为什么要用 B+ 树结构？




## 第二十二题：



## 第二十三题：



## 第二十四题：



























