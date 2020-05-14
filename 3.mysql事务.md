## mysql 事务
mysql中，事务是一个最小的不可分割的工作单元。事务能够保证一个业务的完整性。

比如银行转账：
```
a -> - 100
    update user set money =money-100 where name='a';
b -> + 100
    update user set money =money+100 where name='b';

```
在实际的项目中，假设只用一条SQL语句执行成功，而另外一条执行失败，就会出现数据前后不一致。

因此，多条 SQL 有关联语句，**事务**可能会要求这些 SQL 语句要么同时执行成功，要么就都执行失败。

### 如何控制事务
#### 1. 在mysql中，默认是开启事务的。
```mysql
# 查询事务的自动提交状态
mysql> select @@autocommit;
+--------------+
| @@autocommit |
+--------------+
|            1 |
+--------------+
1 row in set (0.00 sec)
```
> 默认事务开启的作用？  

**自动提交的作用**：当我们执行一条 SQL 语句的时候，其产生的效果就会立即体现出来，且不能**回滚**。

> 什么是回滚？
```mysql
create database bank;

create table user (
    id int primary key,
    name varchar(20),
    money int
);

insert into user value (1,"a",1000);

mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
+----+------+-------+
1 row in set (0.00 sec)

```
可以看到，在执行插入语句后数据立刻生效，原因是 MySQL 中的事务自动将它**提交**到了数据库中。那么所谓**回滚**的意思就是，撤销执行过的所有 SQL 语句，使其回滚到**最后一次提交**数据时的状态。
```mysql

# 事务回滚，撤销sql 语句执行结果
rollback;

mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
+----+------+-------+
1 row in set (0.00 sec)
```
由于所有执行过的 SQL 语句都已经被提交过了，所以数据并没有发生回滚。那如何让数据可以发生回滚？
```mysql 
# 设置关闭自动提交
set autocommit=0;

# 查询提交状态
mysql> select @@autocommit;
+--------------+
| @@autocommit |
+--------------+
|            0 |
+--------------+
1 row in set (0.00 sec)

```
自动提交关闭之后，测试数据回滚
```mysql
insert into user value (2,"b",1000);

mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
|  2 | b    |  1000 |
+----+------+-------+
2 rows in set (0.00 sec)

# 回滚
mysql> rollback;
Query OK, 0 rows affected (0.06 sec)

## 数据被回滚了
mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
+----+------+-------+
1 row in set (0.00 sec)
```
那如何将虚拟的数据真正提交到数据库中？使用 `COMMIT` : 
```mysql
insert into user value (2,"b",1000);

# 手动提交数据（持久性）
mysql> commit; 
Query OK, 0 rows affected (0.00 sec)

# 测试回滚
mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

# 数据未被回滚（回滚失效）
mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
|  2 | b    |  1000 |
+----+------+-------+
2 rows in set (0.00 sec)

```
> **总结**
>
> 1. **自动提交**
>
>    - 查看自动提交状态：`SELECT @@AUTOCOMMIT` ；
>
>    - 设置自动提交状态：`SET AUTOCOMMIT = 0` 。
>
> 2. **手动提交**
>
>    `@@AUTOCOMMIT = 0` 时，使用 `COMMIT` 命令提交事务。
>
> 3. **事务回滚**
>
>    `@@AUTOCOMMIT = 0` 时，使用 `ROLLBACK` 命令回滚事务。


**事务的实际应用**，让我们再回到银行转账项目：
这时假设在转账时发生了意外，就可以使用 `ROLLBACK` 回滚到最后一次提交的状态.
```mysql 
UPDATE user set money = money - 100 WHERE name = 'a';
UPDATE user set money = money + 100 WHERE name = 'b';

mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |   900 |
|  2 | b    |  1100 |
+----+------+-------+
2 rows in set (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
|  2 | b    |  1000 |
+----+------+-------+
2 rows in set (0.00 sec)
```
### 手动开启事务 begin / start transaction
```mysql
mysql> select * from user;     
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
|  2 | b    |  1000 |
+----+------+-------+
2 rows in set (0.00

UPDATE user set money = money - 100 WHERE name = 'a';
UPDATE user set money = money + 100 WHERE name = 'b';

mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |   900 |
|  2 | b    |  1100 |
+----+------+-------+
2 rows in set (0.00 sec)

# rollback 无效
mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |   900 |
|  2 | b    |  1100 |
+----+------+-------+
2 rows in set (0.00 sec)

# 使用begin 或 start transaction 手动开启事务
begin;
UPDATE user set money = money - 100 WHERE name = 'a';
UPDATE user set money = money + 100 WHERE name = 'b';

mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |   800 |
|  2 | b    |  1200 |
+----+------+-------+
2 rows in set (0.00 sec)

# 进行回滚
mysql> rollback;
Query OK, 0 rows affected (0.08 sec)

# 回滚成功
mysql> select * from user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |   900 |
|  2 | b    |  1100 |
+----+------+-------+
```
仍然使用 `COMMIT` 提交数据，提交后无法再发生本次事务的回滚。

## 事务的四大特征 ACID
事务四大特性：
- A 原子性：事务是最小的单位，不可以在分割
- C 一致性：要求同一事务中的SQL语句，必须保证同时成功或同时失败
- I 隔离性：事务1与事务2之间是具有隔离性
- D 持久性：事务一旦结束（commit），就不可以在返回了（rollback）

事务开启：
   1. 修改默认提交，`set autocommit=0`
   2. `begin`
   3. `start transcation`

事务手动提交：`commit`
事务手动回滚：`rollback`


### 事务的隔离性
事务的隔离性分为四种：
1. read uncommitted（读取未提交）
   如果有多个事务，那么任意事务都可以看见其他事务的**未提交数据**。
2. read committed（读取已提交）
3. repeatable read（可被重复度）
4. serializable（串行化）

```mysql
# bank 数据库 user 表
insert into user value (3,"小明",1000);
insert into user value (4,"淘宝店",1000);

mysql> select * from user;                       
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
+----+-----------+-------+
4 rows in set (0.00 sec)
```
如何查看数据库的隔离级别？
```mysql
# mysql 8.x ，其中global表示系统级别，不加表示回话级别
select @@global.transcation_isolation;
# mysql 5.x，其中global表示系统级别，不加表示回话级别
select @@global.tx_isolation;

mysql> select @@global.tx_isolation;
+-----------------------+
| @@global.tx_isolation |
+-----------------------+
| REPEATABLE-READ       |
+-----------------------+
1 row in set, 1 warning (0.00 sec)
```
如何修改隔离级别？
```mysql
set global transaction isolation level read uncommitted;

mysql> select @@global.tx_isolation;
+-----------------------+
| @@global.tx_isolation |
+-----------------------+
| READ-UNCOMMITTED      |
+-----------------------+
1 row in set, 1 warning (0.00 sec)

```
####  读取未提交
```mysql
# 开启一个事务操作数据
# 假设小明在淘宝店买了一双800块钱的鞋子：
start transaction;
update user set money=money-800 where name="小明";
update user set money=money+800 where name="淘宝店";

mysql> select * from user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |   200 |
|  4 | 淘宝店    |  1800 |
+----+-----------+-------+
4 rows in set (0.00 sec)
```
由于小明的转账是在新开启的事务上进行操作的，而该操作的结果是可以被其他事务（另一方的淘宝店）看见的，因此淘宝店的查询结果是正确的，淘宝店确认到账。但就在这时，如果小明在它所处的事务上又执行了 `ROLLBACK` 命令，会发生什么？
```mysql
# 小明所处事务
rollback;

# 转账被回滚了
mysql> select * from user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
+----+-----------+-------+
4 rows in set (0.00 sec)
```
这就是所谓的**脏读**，一个事务读取到另外一个事务还未提交的数据。这在实际开发中是不允许出现的。

#### 读取已提交
```mysql
# 修改隔离级别
set global transaction isolation level read committed;

mysql> SELECT @@GLOBAL.TRANSACTION_ISOLATION;
+--------------------------------+
| @@GLOBAL.TRANSACTION_ISOLATION |
+--------------------------------+
| READ-COMMITTED                 |
+--------------------------------+
1 row in set (0.00 sec)

# bank 数据库 user 表
# 小张：银行会计
start transaction;

mysql> select * from user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
+----+-----------+-------+
4 rows in set (0.00 sec)

# 小张出去上厕所去了...

# 小王 开启新的终端事务，连接数据库
start transaction;
#小王增加一条数据
insert into user values(5,"c",100);
commit;

mysql> select * from user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
|  5 | c         |   100 |
+----+-----------+-------+
5 rows in set (0.00 sec)

# 小张回来后
# 这时小张再求平均值的时候，就会出现计算不相符合的情况：
mysql> select avg(money) from user;
+------------+
| avg(money) |
+------------+
|   820.0000 |
+------------+
1 row in set (0.00 sec)
```
虽然 **READ COMMITTED** 让我们只能读取到其他事务已经提交的数据，但还是会出现问题，就是在读取同一个表的数据时，可能会发生前后不一致的情况。这被称为**不可重复读现象 ( READ COMMITTED 下 )** 。

#### 幻读
将隔离级别设置为 **REPEATABLE READ ( 可被重复读取 )** :
```mysql
# 修改隔离级别
set global transaction isolation level repeatable read;

mysql> SELECT @@GLOBAL.TRANSACTION_ISOLATION;
+--------------------------------+
| @@GLOBAL.TRANSACTION_ISOLATION |
+--------------------------------+
| REPEATABLE-READ                |
+--------------------------------+
1 row in set (0.00 sec)

mysql> select * from user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
|  5 | c         |   100 |
+----+-----------+-------+
5 rows in set (0.00 sec)


# 张全蛋 -- 成都
start transaction;

# 王尼玛 -- 北京
start transaction;

# 张全蛋 -- 成都
insert into user values(6,"d",1000);
commit;

# 王尼玛 -- 北京
## 查询没有6号
mysql> select * from user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
|  5 | c         |   100 |
+----+-----------+-------+
5 rows in set (0.00 sec)

## 但插入6号时报错
mysql> insert into user values(6,"d",1000);
ERROR 1062 (23000): Duplicate entry '6' for key 'PRIMARY'

## 这种现象被称为幻读！
```
当前事务开启后，没提交之前，查询不到，提交后可以被查询到。但是，在==提交之前其他事务被开启==了，那么在这条事务线上，就==不会查询到==当前有操作事务的连接。相当于开辟出一条单独的线程。

无论小张是否执行过 `COMMIT` ，在小王这边，都不会查询到小张的事务记录，而是只会查询到自己所处事务的记录。

这是**因为小王在此之前开启了一个新的事务 ( `START TRANSACTION` ) **，那么**在他的这条新事务的线上，跟其他事务是没有联系的**，也就是说，此时如果其他事务正在操作数据，它是不知道的。

此时插入同一条数据会报错，操作被告知已存在主键为 `6` 的字段。这种现象也被称为**幻读，一个事务提交的数据，不能被其他事务读取到**。


#### 串行化
```mysql
# 修改隔离级别
set global transaction isolation level serializable;

mysql> SELECT @@GLOBAL.TRANSACTION_ISOLATION;
+--------------------------------+
| @@GLOBAL.TRANSACTION_ISOLATION |
+--------------------------------+
| SERIALIZABLE                   |
+--------------------------------+
1 row in set (0.00 sec)

mysql> select * from user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
|  5 | c         |   100 |
|  6 | d         |  1000 |
+----+-----------+-------+
6 rows in set (0.00 sec)

# 张全蛋 -- 成都
start transaction;

# 王尼玛 -- 北京
start transaction;

# 张全蛋 -- 成都
insert into user values(7,"赵铁柱",1000);
commit;

mysql> select * from user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
|  5 | c         |   100 |
|  6 | d         |  1000 |
|  7 | 赵铁柱    |  1000 |
+----+-----------+-------+
7 rows in set (0.00 sec)

# 王尼玛 -- 北京
## 王发现能查询到张的插入
mysql> select * from user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
|  5 | c         |   100 |
|  6 | d         |  1000 |
|  7 | 赵铁柱    |  1000 |
+----+-----------+-------+
7 rows in set (0.00 sec)

# 插入时会被卡住
insert into user values(8,"王小花",1000);

```
此时会发生什么呢？由于现在的隔离级别是 **SERIALIZABLE ( 串行化 )** ，串行化的意思就是：假设把所有的事务都放在一个串行的队列中，那么所有的事务都会按照**固定顺序执行**，执行完一个事务后再继续执行下一个事务的**写入操作** ( **这意味着队列中同时只能执行一个事务的写入操作** ) 。

根据这个解释，王在插入数据时，会出现等待状态，直到小张执行 `COMMIT` 结束它所处的事务，或者出现等待超时。

**==串行化的问题是：性能特差！！！==**

性能：
read-uncommitted > read-committed > repeatable-read > serializable;
隔离级别越高，性能越差！

**mysql 默认隔离级别是 ：`repeatable-read`;**