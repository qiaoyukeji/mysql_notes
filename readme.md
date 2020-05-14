## MySQL 学习笔记

## 关系型数据库


## 一. 如何使用终端操作数据库
### 1.1 常见操作
```mysql
# 登录MySQL
$ mysql -u root -p12345612

# 退出MySQL数据库服务器
exit;

# 显示所有数据库
show databases;

# 进入某个数据库
use 数据库名;

# 创建数据库
create database test;

# 切换进入数据库
use test;

# 查看数据库中的表
mysql> show tables;
Empty set (0.00 sec)


# 创建数据表
CREATE TABLE pet (
    name VARCHAR(20),
    owner VARCHAR(20),
    species VARCHAR(20),
    sex CHAR(1),
    birth DATE,
    death DATE
);

# 查看数据表是否创建成功
mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| pet|
+----------------+
1 row in set (0.00 sec)

# 查看数据表结构
mysql> describe pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | NULL    |       |
| owner   | varchar(20) | YES  |     | NULL    |       |
| species | varchar(20) | YES  |     | NULL    |       |
| sex     | char(1)     | YES  |     | NULL    |       |
| birth   | date        | YES  |     | NULL    |       |
| death   | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
6 rows in set (0.00 sec)

# 查看表中的数据
mysql> select * from pet;
Empty set (0.00 sec)

# 插入单条数据
mysql> insert into pet
    -> value ('puffball', 'Diane', 'hamster', 'f', '1990-03-30', NULL);
Query OK, 1 row affected (0.00 sec)

# 删除数据
mysql> delete from pet where name='puffball'; 
Query OK, 0 rows affected (0.00 sec)

# 修改数据 
update pet set name="zhangsi" where owner ="lisi";

```


> mysql 数据类型
[MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串(字符)类型。](https://www.runoob.com/mysql/mysql-data-types.html)

```mysql
# 总结一下

增加 insert
删除 delete
修改 update
查询 select

```
### 1.2 mysql 建表约束
#### 1.2.1 主键约束
能够唯一确定一张表中的一条记录，给某个字段添加约束后该字段不可重复，且不为空。
```mysql
# 创建带主键约束的user表
create table user(
    id int primary key,
    name varchar(20)
);

mysql> describe user; 
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.04 sec)

insert into user value(1,"张三");

# 联合主键
# 联合主键中的每个字段都不能为空，并且加起来不能和已设置的联合主键重复。
create table user2(
    id int,
    name varchar(20),
    password varchar(20),
    primary key(id,name)
);

mysql> describe user2;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(20) | NO   | PRI | NULL    |       |
| password | varchar(20) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)

```

```mysql
# 自增约束
# 自增约束的主键由系统自动递增分配
create table user3(
    id int primary key auto_increment,
    name varchar(20)
);


# 而外添加主键约束
# 如果忘记设置主键，还可以通过SQL语句设置（两种方式）

alter table user4 add primary key(id);
alter table user4 modify id int primary key;


# 删除主键
alter table user4 drop primary key;

```

#### 1.2.2 唯一约束
约束修饰的字段的值不可以重复
```mysql
# 建表时创建唯一约束
create table user5(
    id int,
    name varchar(20),
    unique(name)
);

# 如果建表时没有设置唯一约束，还可以通过SQL语句设置（两种方式）
alter table user5 add unique(name);
alter table user5 modify name varchar(20) unique;


# 删除唯一约束
alert table user5 drop index name;
```
```mysql
# 总结
# 1. 建表时可以添加约束
# 2. 可以使用alter ... add ...
# 3. 使用alter ... modify ...

# 4. 删除 alter ... drop ...

```


#### 1.2.3 非空约束
修饰的字段不能为空 null
```mysql
create table user6 (
    id int ,
    name varchar(20) not null
);

mysql> describe user6;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(20) | NO   |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.04 sec)


# 移除非空约束
alter table user6 modefy name varchar(20);

```
#### 1.2.4 默认约束
当插入字段值时，如果没有传值，就会使用默认值
```mysql

create table user7(
    id int,
    name varchar(20),
    age int default 10
);

mysql> desc user7;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| age   | int(11)     | YES  |     | 10      |       |
+-------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)

# 移除默认约束
alter table user7 modify age int;

```


#### 1.2.5 外键约束
涉及两个表：主表，附表。

```mysql
# 班级表
create table classes(
    id int primary key,
    name varchar(20)
);

# 学生表
# 学生表的class_id 与班级表 id 绑定
create table students(
    id int primary key,
    name varchar(20),
    class_id int ,
    foreign key(class_id) references classes(id)
);

insert into classes values (1,"一班");
insert into classes values (2,"二班");
insert into classes values (3,"三班");
insert into classes values (4,"四班");

insert into students values(1001,"张三",1);
insert into students values(1002,"张三",2);
insert into students values(1003,"张三",3);
insert into students values(1004,"张三",4);

# 1. 主表 classes 中没有的数据值，在副表中是不可以使用的。
# 2. 主表的记录被副表引用时，主表是不可以被删除。

```
### 1.3 数据库的三大范式
#### 1NF
数据表中所有字段都是不可分割的原子值。
只要字段值还可以继续拆分，就不满足第一范式。

范式设计得越详细，对某些实际操作可能会更好，但并非都有好处，需要对项目的实际情况进行设定。

#### 2NF
在满足第一范式的前提下，其他列都必须完全依赖于主键。
如果出现不完全依赖，只可能发生在联合主键的情况下。

```mysql
# 订单表
create table myorder(
    product_id int,
    customer_id int,
    product_name varchar(20),
    customer_name varchar(20),
    primary key(product_id,customer_id)
);
```
实际上，在这张订单表中，`product_name` 只依赖于 `product_id` ，`customer_name` 只依赖于 `customer_id` 。也就是说，`product_name` 和 `customer_id` 是没用关系的，`customer_name` 和 `product_id` 也是没有关系的。

这就不满足第二范式：其他列都必须完全依赖于主键列！

```mysql
create table myorder(
    order_id int primary key,
    product_id int,
    customer_id int
);

create table product(
    id int primary key,
    name varchar(20)
);

create table customer(
    id int primary key,
    name varchar(20)
);
```
拆分之后，`myorder` 表中的 `product_id` 和 `customer_id` 完全依赖于 `order_id` 主键，而 `product` 和 `customer` 表中的其他字段又完全依赖于主键。满足了第二范式的设计！

#### 3NF
在满足第二范式的前提下，除了主键列之外的其他列不能有传递依赖关系。
```mysql
create table myorder(
    order_id int primary key,
    product_id int,
    customer_id int,
    customer_phone varchar(15)
);
```
表中的 `customer_phone` 有可能依赖于 `order_id` 、 `customer_id` 两列，也就不满足了第三范式的设计：其他列之间不能有传递依赖关系。

```mysql
create table myorder(
    order_id int primary key,
    product_id int,
    customer_id int
);

create table customer(
    id int primary key,
    name varchar(20),
    customer_phone varchar(15)
);
```
修改后就不存在其他列之间的传递依赖关系，其他列都只依赖于主键列，满足了第三范式的设计！

## 二. 查询练习
### 2.1 准备数据

1. 学生表
> student
> > 学号
> 姓名
> 性别
> 出生年月
> 所在班级


2. 课程表
> course 
> > 课程号
> 课程名
> 教师编号


3. 成绩表
> score
> > 学号
> 课程号
> 成绩


4. 教师表
> teacher
> > 教师编号
> 教师名
> 教师性别
> 出生年月
> 职称
> 所在部门

```mysql
# 创建新的数据库
create database select_test;

# 进入数据库
use select_test;

# 创建学生表
create table student(
    sno varchar(20) primary key,
    sname varchar(20) not null,
    ssex varchar(10) not null,
    sbirthday datetime,
    class varchar(20)
);

# 创建教师表
create table teacher(
    tno varchar(20) primary key,
    tname varchar(20) not null,
    tsex varchar(10) not null,
    tbirthday datetime,
    profession varchar(20) not null,
    department varchar(20) not null
);

# 创建课程表
create table course(
    cno varchar(20) primary key,
    cname varchar(20) not null,
    tno varchar(20) not null,
    foreign key(tno) references teacher(tno)
);

# 创建成绩表
create table score (
    sno varchar(20) not null,
    cno varchar(20) not null,
    dgree decimal,
    foreign key(sno) references student(sno),
    foreign key(cno) references course(cno),
    primary key(sno,cno)
);


# 添加学生表数据
INSERT INTO student VALUES('101', '曾华', '男', '1977-09-01', '95033');
INSERT INTO student VALUES('102', '匡明', '男', '1975-10-02', '95031');
INSERT INTO student VALUES('103', '王丽', '女', '1976-01-23', '95033');
INSERT INTO student VALUES('104', '李军', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('105', '王芳', '女', '1975-02-10', '95031');
INSERT INTO student VALUES('106', '陆军', '男', '1974-06-03', '95031');
INSERT INTO student VALUES('107', '王尼玛', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('108', '张全蛋', '男', '1975-02-10', '95031');
INSERT INTO student VALUES('109', '赵铁柱', '男', '1974-06-03', '95031');


# 添加教师表数据
INSERT INTO teacher VALUES('804', '李诚', '男', '1958-12-02', '副教授', '计算机系');
INSERT INTO teacher VALUES('856', '张旭', '男', '1969-03-12', '讲师', '电子工程系');
INSERT INTO teacher VALUES('825', '王萍', '女', '1972-05-05', '助教', '计算机系');
INSERT INTO teacher VALUES('831', '刘冰', '女', '1977-08-14', '助教', '电子工程系');

# 添加课程数据
INSERT INTO course VALUES('3-105', '计算机导论', '825');
INSERT INTO course VALUES('3-245', '操作系统', '804');
INSERT INTO course VALUES('6-166', '数字电路', '856');
INSERT INTO course VALUES('9-888', '高等数学', '831');

# 添加成绩表
INSERT INTO score VALUES('103', '3-105', '92');
INSERT INTO score VALUES('103', '3-245', '86');
INSERT INTO score VALUES('103', '6-166', '85');
INSERT INTO score VALUES('105', '3-105', '88');
INSERT INTO score VALUES('105', '3-245', '75');
INSERT INTO score VALUES('105', '6-166', '79');
INSERT INTO score VALUES('109', '3-105', '76');
INSERT INTO score VALUES('109', '3-245', '68');
INSERT INTO score VALUES('109', '6-166', '81');

```
> 查询练习

```mysql
# 1. 查询student 表的所有记录。
mysql> select * from student;
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 102 | 匡明      | 男   | 1975-10-02 00:00:00 | 95031 |
| 103 | 王丽      | 女   | 1976-01-23 00:00:00 | 95033 |
| 104 | 李军      | 男   | 1976-02-20 00:00:00 | 95033 |
| 105 | 王芳      | 女   | 1975-02-10 00:00:00 | 95031 |
| 106 | 陆军      | 男   | 1974-06-03 00:00:00 | 95031 |
| 107 | 王尼玛    | 男   | 1976-02-20 00:00:00 | 95033 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
| 109 | 赵铁柱    | 男   | 1974-06-03 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
8 rows in set (0.00 sec)

# 2. 查询student表中所有记录的sname 、ssex和class 列。
mysql> select sname,ssex,class from student;
+-----------+------+-------+
| sname     | ssex | class |
+-----------+------+-------+
| 匡明      | 男   | 95031 |
| 王丽      | 女   | 95033 |
| 李军      | 男   | 95033 |
| 王芳      | 女   | 95031 |
| 陆军      | 男   | 95031 |
| 王尼玛    | 男   | 95033 |
| 张全蛋    | 男   | 95031 |
| 赵铁柱    | 男   | 95031 |
+-----------+------+-------+
8 rows in set (0.00 sec)

# 3. 查询教师所有单位及不重复的department 列
# 去重查询 使用关键字 distinct
mysql> select distinct department from teacher;
+-----------------+
| department      |
+-----------------+
| 计算机系        |
| 电子工程系      |
+-----------------+
2 rows in set (0.00 sec)

# 4. 查询score表中成绩在60到80之间的所有记录
# 查询区间 where 条件 between...and... 区间
mysql> select * from score where dgree between 60 and 80;
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 105 | 3-245 |    75 |
| 105 | 6-166 |    79 |
| 109 | 3-105 |    76 |
| 109 | 3-245 |    68 |
+-----+-------+-------+
4 rows in set (0.00 sec)

mysql> select * from score where dgree > 60 and dgree < 80;
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 105 | 3-245 |    75 |
| 105 | 6-166 |    79 |
| 109 | 3-105 |    76 |
| 109 | 3-245 |    68 |
+-----+-------+-------+
4 rows in set (0.04 sec)

# 5. 查询score 表中成绩为85、86或88的记录
# 或者关系查询 in
mysql> select * from score where dgree in(85,86,88);
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 103 | 3-245 |    86 |
| 103 | 6-166 |    85 |
| 105 | 3-105 |    88 |
+-----+-------+-------+
3 rows in set (0.00 sec)

# 6. 查询student表中“95031” 班或者性别为“女” 的同学记录。
# 或者关系  or
mysql> select * from student where class='95031' or ssex="女";
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 102 | 匡明      | 男   | 1975-10-02 00:00:00 | 95031 |
| 103 | 王丽      | 女   | 1976-01-23 00:00:00 | 95033 |
| 105 | 王芳      | 女   | 1975-02-10 00:00:00 | 95031 |
| 106 | 陆军      | 男   | 1974-06-03 00:00:00 | 95031 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
| 109 | 赵铁柱    | 男   | 1974-06-03 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
6 rows in set (0.00 sec)

# 7. 以class 班级降序的方式查询 student 表中所有记录。
# 升序(默认):asc   降序:desc
mysql> select * from student order by class desc;
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 103 | 王丽      | 女   | 1976-01-23 00:00:00 | 95033 |
| 104 | 李军      | 男   | 1976-02-20 00:00:00 | 95033 |
| 107 | 王尼玛    | 男   | 1976-02-20 00:00:00 | 95033 |
| 102 | 匡明      | 男   | 1975-10-02 00:00:00 | 95031 |
| 105 | 王芳      | 女   | 1975-02-10 00:00:00 | 95031 |
| 106 | 陆军      | 男   | 1974-06-03 00:00:00 | 95031 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
| 109 | 赵铁柱    | 男   | 1974-06-03 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
8 rows in set (0.00 sec)

# 8. 以 cno 升序、dgree降序查询score表中的所有记录。
mysql> select * from score order by cno asc,dgree desc;
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 103 | 3-105 |    92 |
| 105 | 3-105 |    88 |
| 109 | 3-105 |    76 |
| 103 | 3-245 |    86 |
| 105 | 3-245 |    75 |
| 109 | 3-245 |    68 |
| 103 | 6-166 |    85 |
| 109 | 6-166 |    81 |
| 105 | 6-166 |    79 |
+-----+-------+-------+
9 rows in set (0.00 sec)

# 9. 查询“95031” 班的学生人数
# 统计 count
mysql> select count(*) from student where class="95031";
+----------+
| count(*) |
+----------+
|        5 |
+----------+
1 row in set (0.00 sec)

# 10. 查询score 表中的最高分学生学号和课程号。(子查询或排序)

## 子查询
### 第一步 找到最高分 
select max(dgree) from score
### 第二步 找最高分的sno和cno
mysql> select sno,cno from score where dgree=(select max(dgree) from score);
+-----+-------+
| sno | cno   |
+-----+-------+
| 103 | 3-105 |
+-----+-------+
1 row in set (0.00 sec)

## 排序查询(了解即可)
## limit r,n; 表示从r行开始，查 n 条数据
mysql> select sno,cno from score order by dgree desc limit 0,1;
+-----+-------+
| sno | cno   |
+-----+-------+
| 103 | 3-105 |
+-----+-------+
1 row in set (0.00 sec)

```

```mysql
# 11. 查询每一门课的平均成绩
select * from course;
## avg() 平均
mysql> select avg(dgree) from score where cno="3-105";
+------------+
| avg(dgree) |
+------------+
|    85.3333 |
+------------+
1 row in set (0.05 sec)

## group by 分组
mysql> select cno,avg(dgree) from score group by cno;
+-------+------------+
| cno   | avg(dgree) |
+-------+------------+
| 3-105 |    85.3333 |
| 3-245 |    76.3333 |
| 6-166 |    81.6667 |
+-------+------------+
3 rows in set (0.00 sec)


# 12. 查询score表中至少有两名学生选修的并以3开头的课程的平均分。
## having 表示持有
## like 表示模糊查询
mysql> select cno,avg(dgree) from score group by cno having count(cno)>=2 and cno like "3%";
+-------+------------+
| cno   | avg(dgree) |
+-------+------------+
| 3-105 |    85.3333 |
| 3-245 |    76.3333 |
+-------+------------+
2 rows in set (0.00 sec)


# 13. 查询分数大于70小于90的sno列

mysql> select sno,dgree from score where dgree>70 and dgree<90;
+-----+-------+
| sno | dgree |
+-----+-------+
| 103 |    86 |
| 103 |    85 |
| 105 |    88 |
| 105 |    75 |
| 105 |    79 |
| 109 |    76 |
| 109 |    81 |
+-----+-------+
7 rows in set (0.00 sec)

mysql> select sno,dgree from score where dgree between 70 and 90;
+-----+-------+
| sno | dgree |
+-----+-------+
| 103 |    86 |
| 103 |    85 |
| 105 |    88 |
| 105 |    75 |
| 105 |    79 |
| 109 |    76 |
| 109 |    81 |
+-----+-------+
7 rows in set (0.00 sec)

# 14. 查询所有学生的sname、cno和dgree列。
mysql> select sname,sno from student;
+-----------+-----+
| sname     | sno |
+-----------+-----+
| 匡明      | 102 |
| 王丽      | 103 |
| 李军      | 104 |
| 王芳      | 105 |
| 陆军      | 106 |
| 王尼玛    | 107 |
| 张全蛋    | 108 |
| 赵铁柱    | 109 |
+-----------+-----+
8 rows in set (0.00 sec)

mysql> select cno,dgree,sno from score;
+-------+-------+-----+
| cno   | dgree | sno |
+-------+-------+-----+
| 3-105 |    92 | 103 |
| 3-245 |    86 | 103 |
| 6-166 |    85 | 103 |
| 3-105 |    88 | 105 |
| 3-245 |    75 | 105 |
| 6-166 |    79 | 105 |
| 3-105 |    76 | 109 |
| 3-245 |    68 | 109 |
| 6-166 |    81 | 109 |
+-------+-------+-----+
9 rows in set (0.00 sec)

## 进行组合查询(通过sno连接起两表的联系)
mysql> select sname,cno,dgree from student,score where student.sno=score.sno;
+-----------+-------+-------+
| sname     | cno   | dgree |
+-----------+-------+-------+
| 王丽      | 3-105 |    92 |
| 王丽      | 3-245 |    86 |
| 王丽      | 6-166 |    85 |
| 王芳      | 3-105 |    88 |
| 王芳      | 3-245 |    75 |
| 王芳      | 6-166 |    79 |
| 赵铁柱    | 3-105 |    76 |
| 赵铁柱    | 3-245 |    68 |
| 赵铁柱    | 6-166 |    81 |
+-----------+-------+-------+
9 rows in set (0.00 sec)

# 15. 查询所有学生的sno、cname和dgree列

mysql> select cno,cname from course;
+-------+-----------------+
| cno   | cname           |
+-------+-----------------+
| 3-105 | 计算机导论      |
| 3-245 | 操作系统        |
| 6-166 | 数字电路        |
| 9-888 | 高等数学        |
+-------+-----------------+
4 rows in set (0.00 sec)


mysql> select cno,sno,dgree from score;
+-------+-----+-------+
| cno   | sno | dgree |
+-------+-----+-------+
| 3-105 | 103 |    92 |
| 3-245 | 103 |    86 |
| 6-166 | 103 |    85 |
| 3-105 | 105 |    88 |
| 3-245 | 105 |    75 |
| 6-166 | 105 |    79 |
| 3-105 | 109 |    76 |
| 3-245 | 109 |    68 |
| 6-166 | 109 |    81 |
+-------+-----+-------+
9 rows in set (0.00 sec)

## 进行组合查询(通过cno 连接起两表的联系)
mysql> select sno,cname,dgree from course,score where score.cno=course.cno;
+-----+-----------------+-------+
| sno | cname           | dgree |
+-----+-----------------+-------+
| 103 | 计算机导论      |    92 |
| 103 | 操作系统        |    86 |
| 103 | 数字电路        |    85 |
| 105 | 计算机导论      |    88 |
| 105 | 操作系统        |    75 |
| 105 | 数字电路        |    79 |
| 109 | 计算机导论      |    76 |
| 109 | 操作系统        |    68 |
| 109 | 数字电路        |    81 |
+-----+-----------------+-------+
9 rows in set (0.00 sec)

# 16. 查询所有学生的sname、cname和dgree
## sname -> student
## cname -> course
## dgree -> score
## 通过cno和sno将三表联系起来

mysql> select sname,cname,dgree from student,course,score where student.sno=score.sno and score.cno=course.cno;
+-----------+-----------------+-------+
| sname     | cname           | dgree |
+-----------+-----------------+-------+
| 王丽      | 计算机导论      |    92 |
| 王丽      | 操作系统        |    86 |
| 王丽      | 数字电路        |    85 |
| 王芳      | 计算机导论      |    88 |
| 王芳      | 操作系统        |    75 |
| 王芳      | 数字电路        |    79 |
| 赵铁柱    | 计算机导论      |    76 |
| 赵铁柱    | 操作系统        |    68 |
| 赵铁柱    | 数字电路        |    81 |
+-----------+-----------------+-------+
9 rows in set (0.00 sec)

# 17. 查询“95031”班学生每门课的平均成绩。

select * from student where class="95031";

mysql> select * from score where sno in (select sno from student where class="95031");
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 105 | 3-105 |    88 |
| 105 | 3-245 |    75 |
| 105 | 6-166 |    79 |
| 109 | 3-105 |    76 |
| 109 | 3-245 |    68 |
| 109 | 6-166 |    81 |
+-----+-------+-------+
6 rows in set (0.00 sec)

mysql> select cno,avg(dgree) from score where sno in (select sno from student where class="95031") group by cno;
+-------+------------+
| cno   | avg(dgree) |
+-------+------------+
| 3-105 |    82.0000 |
| 3-245 |    71.5000 |
| 6-166 |    80.0000 |
+-------+------------+
3 rows in set (0.10 sec)


# 18. 查询选修“3-105”课程的成绩高于109号同学“3-105”成绩的所有同学的记录。
mysql> select dgree from score where sno="109" and cno="3-105";
+-------+
| dgree |
+-------+
|    76 |
+-------+
1 row in set (0.00 sec)


mysql> select * from score where dgree > (select dgree from score where sno="109" and cno="3-105") and cno="3-105";
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 103 | 3-105 |    92 |
| 105 | 3-105 |    88 |
+-----+-------+-------+
2 rows in set (0.06 sec)


# 19. 查询成绩高于学号为109、课程号为“3-105”的成绩的所有记录。
mysql> select * from score where dgree > (select dgree from score where sno="109" and cno="3-105");
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 103 | 3-105 |    92 |
| 103 | 3-245 |    86 |
| 103 | 6-166 |    85 |
| 105 | 3-105 |    88 |
| 105 | 6-166 |    79 |
| 109 | 6-166 |    81 |
+-----+-------+-------+
6 rows in set (0.00 sec)

# 20. 查询和学号为108、101的同学同年出生的所有学生的sno、sname和sbirthday。

mysql> select year(sbirthday) from student where sno in (108,101);
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 101 | 曾华      | 男   | 1977-09-01 00:00:00 | 95033 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
2 rows in set (0.00 sec)

mysql> select year(sbirthday) from student where sno in (108,101);
+-----------------+
| year(sbirthday) |
+-----------------+
|            1977 |
|            1975 |
+-----------------+
2 rows in set (0.04 sec)

mysql> select * from student where year(sbirthday) in (select year(sbirthday) from student where sno in (108,101));
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 101 | 曾华      | 男   | 1977-09-01 00:00:00 | 95033 |
| 102 | 匡明      | 男   | 1975-10-02 00:00:00 | 95031 |
| 105 | 王芳      | 女   | 1975-02-10 00:00:00 | 95031 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
4 rows in set (0.00 sec)

```
#### 多层嵌套子查询
```mysql
# 21. 查询 `'张旭'` 教师任课的学生成绩表。
## 首先找到教师编号：
mysql> select tno from teacher where tname="张旭"; 
+-----+
| tno |
+-----+
| 856 |
+-----+
1 row in set (0.00 sec)

## 通过 course 表找到该教师课程号：
mysql> select cno from course where tno=(select tno from teacher where tname="张旭");
+-------+
| cno   |
+-------+
| 6-166 |
+-------+
1 row in set (0.00 sec)

## 通过筛选出的课程号查询成绩表：
mysql> select * from score where cno=(select cno from course where tno=(select tno from teacher where tname="张旭")) ;
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 103 | 6-166 |    85 |
| 105 | 6-166 |    79 |
| 109 | 6-166 |    81 |
+-----+-------+-------+
3 rows in set (0.00 sec)

```
#### 多表查询
```mysql
# 22. 查询某选修课程多于5个同学的教师姓名

## 补充数据
INSERT INTO score VALUES('101', '3-105', '90');
INSERT INTO score VALUES('102', '3-105', '91');
INSERT INTO score VALUES('104', '3-105', '89');

## 在成绩表中查出选修课程多于5个同学的课程号
mysql> select cno from score group by cno having count(cno)>5;
+-------+
| cno   |
+-------+
| 3-105 |
+-------+
1 row in set (0.00 sec)

## 通过课程号在课程表中找出教师编号
mysql> select tno from course where cno=(select cno from score group by cno having count(cno)>5);
+-----+
| tno |
+-----+
| 825 |
+-----+
1 row in set (0.00 sec)

## 然后在教师表中找到教师名
mysql> select tname from teacher where tno=(select tno from course where cno=(select cno from score group by cno having count(cno)>5));
+--------+
| tname  |
+--------+
| 王萍   |
+--------+
1 row in set (0.00 sec)

# 23. 查询95033和95031班的全体学生记录
mysql> select * from student where class in(95033,95031);
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 101 | 曾华      | 男   | 1977-09-01 00:00:00 | 95033 |
| 102 | 匡明      | 男   | 1975-10-02 00:00:00 | 95031 |
| 103 | 王丽      | 女   | 1976-01-23 00:00:00 | 95033 |
| 104 | 李军      | 男   | 1976-02-20 00:00:00 | 95033 |
| 105 | 王芳      | 女   | 1975-02-10 00:00:00 | 95031 |
| 106 | 陆军      | 男   | 1974-06-03 00:00:00 | 95031 |
| 107 | 王尼玛    | 男   | 1976-02-20 00:00:00 | 95033 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
| 109 | 赵铁柱    | 男   | 1974-06-03 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
9 rows in set (0.00 sec)

# 24. 查询存在有85分以上成绩的cno。
mysql> select cno from score where dgree > 85;
+-------+
| cno   |
+-------+
| 3-105 |
| 3-105 |
| 3-105 |
| 3-245 |
| 3-105 |
| 3-105 |
+-------+
6 rows in set (0.00 sec)

```

```mysql
# 25. 查询出 计算机系 教师所教课程的成绩表

## 先查出计算机系老师的tno
mysql> select tno from teacher where department = "计算机系";
+-----+
| tno |
+-----+
| 804 |
| 825 |
+-----+
2 rows in set (0.00 sec)

## 根据教师的tno 查询课程表中cno
mysql> select cno from course where tno in (select tno from teacher where department = "计算机系");
+-------+
| cno   |
+-------+
| 3-245 |
| 3-105 |
+-------+
2 rows in set (0.00 sec)

## 根据cno 查询成绩表
mysql> select * from score where cno in(select cno from course where tno in (select tno from teacher where department = "计算机系"));
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 103 | 3-245 |    86 |
| 105 | 3-245 |    75 |
| 109 | 3-245 |    68 |
| 101 | 3-105 |    90 |
| 102 | 3-105 |    91 |
| 103 | 3-105 |    92 |
| 104 | 3-105 |    89 |
| 105 | 3-105 |    88 |
| 109 | 3-105 |    76 |
+-----+-------+-------+
9 rows in set (0.00 sec)

```
#### UNION 与 NOT IN 的使用
```mysql
# 26. 查询 计算机系 和 电子工程系 不同职称的教师的tname和profession

## not 代表逻辑非
mysql> select * from teacher where department="计算机系" and profession not in (select profession from teacher where department="电子工程系");
+-----+--------+------+---------------------+------------+--------------+
| tno | tname  | tsex | tbirthday           | profession | department   |
+-----+--------+------+---------------------+------------+--------------+
| 804 | 李诚   | 男   | 1958-12-02 00:00:00 | 副教授     | 计算机系     |
+-----+--------+------+---------------------+------------+--------------+
1 row in set (0.00 sec)

mysql> select * from teacher where department="电子工程系" and profession not in (select profession from teacher where department="计算机系");
+-----+--------+------+---------------------+------------+-----------------+
| tno | tname  | tsex | tbirthday           | profession | department      |
+-----+--------+------+---------------------+------------+-----------------+
| 856 | 张旭   | 男   | 1969-03-12 00:00:00 | 讲师       | 电子工程系      |
+-----+--------+------+---------------------+------------+-----------------+
1 row in set (0.00 sec)

## union 合并两个集合求并集
mysql> select tname,profession from teacher where department="电子工程系" and profession not in (select profession from teacher where department="计算机系") union select tname,profession from teacher where department="计算机系" and profession not in (select profession from teacher where department="电子工程系");
+--------+------------+
| tname  | profession |
+--------+------------+
| 张旭   | 讲师       |
| 李诚   | 副教授     |
+--------+------------+
2 rows in set (0.00 sec)
```
#### any 表示至少一个
```mysql
# 27. 查询课程为 3-105 且成绩至少高于 3-245 的 同学的cno、sno和dgree，并且按dgree 从低到高排序。
## any 表示至少一个
mysql> select * from score where cno="3-245";
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 103 | 3-245 |    86 |
| 105 | 3-245 |    75 |
| 109 | 3-245 |    68 |
+-----+-------+-------+
3 rows in set (0.00 sec)

mysql> select * from score where cno="3-105" and dgree> any(select dgree from score where cno="3-245") order by dgree desc;
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 103 | 3-105 |    92 |
| 102 | 3-105 |    91 |
| 101 | 3-105 |    90 |
| 104 | 3-105 |    89 |
| 105 | 3-105 |    88 |
| 109 | 3-105 |    76 |
+-----+-------+-------+
6 rows in set (0.00 sec)

```
#### all 表示所有
```mysql 
# 28. 查询课程为 3-105 且成绩高于 3-245 的 同学的cno、sno和dgree。
## all 表示所有
mysql> select * from score where cno="3-105" and dgree> all(select dgree from score where cno="3-245");
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 101 | 3-105 |    90 |
| 102 | 3-105 |    91 |
| 103 | 3-105 |    92 |
| 104 | 3-105 |    89 |
| 105 | 3-105 |    88 |
+-----+-------+-------+
5 rows in set (0.00 sec)

```
#### as 取别名
```mysql
# 29. 查询所有教师和同学的name，sex和birthday
## as 取别名
mysql> select tname as name,tsex as sex,tbirthday as birthday from teacher union select sname as name,ssex as sex,sbirthday as birthday from student;
+-----------+-----+---------------------+
| name      | sex | birthday            |
+-----------+-----+---------------------+
| 李诚      | 男  | 1958-12-02 00:00:00 |
| 王萍      | 女  | 1972-05-05 00:00:00 |
| 刘冰      | 女  | 1977-08-14 00:00:00 |
| 张旭      | 男  | 1969-03-12 00:00:00 |
| 曾华      | 男  | 1977-09-01 00:00:00 |
| 匡明      | 男  | 1975-10-02 00:00:00 |
| 王丽      | 女  | 1976-01-23 00:00:00 |
| 李军      | 男  | 1976-02-20 00:00:00 |
| 王芳      | 女  | 1975-02-10 00:00:00 |
| 陆军      | 男  | 1974-06-03 00:00:00 |
| 王尼玛    | 男  | 1976-02-20 00:00:00 |
| 张全蛋    | 男  | 1975-02-10 00:00:00 |
| 赵铁柱    | 男  | 1974-06-03 00:00:00 |
+-----------+-----+---------------------+
13 rows in set (0.00 sec)


# 30. 查询所有 女 教师和 女 学生的name 、sex和birthday
mysql> select tname as name,tsex as sex,tbirthday as birthday from teacher where tsex="女" union select sname as name,ssex as sex,sbirthday as birthday from student where ssex="女";
+--------+-----+---------------------+
| name   | sex | birthday            |
+--------+-----+---------------------+
| 王萍   | 女  | 1972-05-05 00:00:00 |
| 刘冰   | 女  | 1977-08-14 00:00:00 |
| 王丽   | 女  | 1976-01-23 00:00:00 |
| 王芳   | 女  | 1975-02-10 00:00:00 |
+--------+-----+---------------------+
4 rows in set (0.00 sec)

```
#### 复制表的数据作为条件查询 (一张表当两张表用)
```mysql
# 31. 查询某课程成绩比该课程平均成绩低的成绩表。
## 先查询平均成绩
mysql> select cno,avg(dgree) from score group by cno; 
+-------+------------+
| cno   | avg(dgree) |
+-------+------------+
| 3-105 |    87.6667 |
| 3-245 |    76.3333 |
| 6-166 |    81.6667 |
+-------+------------+
3 rows in set (0.00 sec)

## 然后查询表中成绩小于平均值的
## 下表中 a b 均为 score表，将一张表当两张表用
mysql> select * from score a where dgree<(select avg(dgree) from score b where a.cno=b.cno);
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 105 | 3-245 |    75 |
| 105 | 6-166 |    79 |
| 109 | 3-105 |    76 |
| 109 | 3-245 |    68 |
| 109 | 6-166 |    81 |
+-----+-------+-------+
5 rows in set (0.00 sec)

```
#### 子查询
```mysql
# 32. 查询所有任课老师的Tname和department。
select * from teacher;
## 在课程表里有课程的老师编号
mysql> select tno from course;
+-----+
| tno |
+-----+
| 804 |
| 825 |
| 831 |
| 856 |
+-----+
4 rows in set (0.08 sec)

## 然后在教师表中匹配课程表中有课程的教师编号
mysql> select tname,department from teacher where tno in (select tno from course);
+--------+-----------------+
| tname  | department      |
+--------+-----------------+
| 李诚   | 计算机系        |
| 王萍   | 计算机系        |
| 刘冰   | 电子工程系      |
| 张旭   | 电子工程系      |
+--------+-----------------+
4 rows in set (0.00 sec)

```
#### 条件加分组筛选
```mysql
# 33. 查询至少有两名男生的班号

mysql> select * from student ;
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 101 | 曾华      | 男   | 1977-09-01 00:00:00 | 95033 |
| 102 | 匡明      | 男   | 1975-10-02 00:00:00 | 95031 |
| 103 | 王丽      | 女   | 1976-01-23 00:00:00 | 95033 |
| 104 | 李军      | 男   | 1976-02-20 00:00:00 | 95033 |
| 105 | 王芳      | 女   | 1975-02-10 00:00:00 | 95031 |
| 106 | 陆军      | 男   | 1974-06-03 00:00:00 | 95031 |
| 107 | 王尼玛    | 男   | 1976-02-20 00:00:00 | 95033 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
| 109 | 赵铁柱    | 男   | 1974-06-03 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
9 rows in set (0.00 sec)

## 只查询性别为男，然后按 class 分组，并限制 class 行大于 1。
mysql> select class from student where ssex="男" group by class having count(*)>1;
+-------+
| class |
+-------+
| 95031 |
| 95033 |
+-------+
2 rows in set (0.00 sec)

```
#### not like 模糊查询取反
```mysql
# 34. 查询student 表中不行 王 的同学记录
select * from student;
## not like 模糊查询取反
mysql> select * from student where sname not like "王%";
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 101 | 曾华      | 男   | 1977-09-01 00:00:00 | 95033 |
| 102 | 匡明      | 男   | 1975-10-02 00:00:00 | 95031 |
| 104 | 李军      | 男   | 1976-02-20 00:00:00 | 95033 |
| 106 | 陆军      | 男   | 1974-06-03 00:00:00 | 95031 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
| 109 | 赵铁柱    | 男   | 1974-06-03 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
6 rows in set (0.05 sec)

```
#### year 函数与 now 函数
```mysql
# 35. 查询student 表中每个学生的姓名和年龄
select * from student;

## 使用函数 YEAR(NOW()) 计算出当前年份
mysql> select year(now());
+-------------+
| year(now()) |
+-------------+
|        2020 |
+-------------+

mysql> select year(sbirthday) from student;
+-----------------+
| year(sbirthday) |
+-----------------+
|            1977 |
|            1975 |
|            1976 |
|            1976 |
|            1975 |
|            1974 |
|            1976 |
|            1975 |
|            1974 |
+-----------------+
9 rows in set (0.00 sec)


mysql> select sname,year(now())-year(sbirthday) as age from student;
+-----------+------+
| sname     | age  |
+-----------+------+
| 曾华      |   43 |
| 匡明      |   45 |
| 王丽      |   44 |
| 李军      |   44 |
| 王芳      |   45 |
| 陆军      |   46 |
| 王尼玛    |   44 |
| 张全蛋    |   45 |
| 赵铁柱    |   46 |
+-----------+------+
9 rows in set (0.05 sec)

``` 
#### max 和 min 函数
```mysql
# 36. 查询studnet 表中最大和最小的sbirthday 日期值

mysql> select sbirthday from student order by sbirthday;
+---------------------+
| sbirthday           |
+---------------------+
| 1974-06-03 00:00:00 |
| 1974-06-03 00:00:00 |
| 1975-02-10 00:00:00 |
| 1975-02-10 00:00:00 |
| 1975-10-02 00:00:00 |
| 1976-01-23 00:00:00 |
| 1976-02-20 00:00:00 |
| 1976-02-20 00:00:00 |
| 1977-09-01 00:00:00 |
+---------------------+
9 rows in set (0.00 sec)

# max() min()
select max(sbirthday),min(sbirthday) from student;
```
### 多字段排序
```mysql
# 37. 以班号和年龄从大到小的顺序查询student 表中的全部记录。
mysql> select * from student order by class desc,sbirthday;
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 103 | 王丽      | 女   | 1976-01-23 00:00:00 | 95033 |
| 104 | 李军      | 男   | 1976-02-20 00:00:00 | 95033 |
| 107 | 王尼玛    | 男   | 1976-02-20 00:00:00 | 95033 |
| 101 | 曾华      | 男   | 1977-09-01 00:00:00 | 95033 |
| 106 | 陆军      | 男   | 1974-06-03 00:00:00 | 95031 |
| 109 | 赵铁柱    | 男   | 1974-06-03 00:00:00 | 95031 |
| 105 | 王芳      | 女   | 1975-02-10 00:00:00 | 95031 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
| 102 | 匡明      | 男   | 1975-10-02 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
9 rows in set (0.00 sec)

```
#### 子查询
```mysql
# 38. 查询男教师及其所上的课程

## 先查询男教师编号
mysql> select * from teacher where tsex="男";
+-----+--------+------+---------------------+------------+-----------------+
| tno | tname  | tsex | tbirthday           | profession | department      |
+-----+--------+------+---------------------+------------+-----------------+
| 804 | 李诚   | 男   | 1958-12-02 00:00:00 | 副教授     | 计算机系        |
| 856 | 张旭   | 男   | 1969-03-12 00:00:00 | 讲师       | 电子工程系      |
+-----+--------+------+---------------------+------------+-----------------+
2 rows in set (0.00 sec)

## 根据老师编号在课程表中查找课程
mysql> select * from course where tno in (select tno from teacher where tsex="男");
+-------+--------------+-----+
| cno   | cname        | tno |
+-------+--------------+-----+
| 3-245 | 操作系统     | 804 |
| 6-166 | 数字电路     | 856 |
+-------+--------------+-----+
2 rows in set (0.00 sec)

```
#### max 查询与子查询
```mysql
# 39. 查询最高分同学的sno、cno的dgree

mysql> select max(dgree) from score ;
+------------+
| max(dgree) |
+------------+
|         92 |
+------------+
1 row in set (0.07 sec)

mysql> select * from score where dgree =(select max(dgree) from score);
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 103 | 3-105 |    92 |
+-----+-------+-------+
1 row in set (0.00 sec)

# 40. 查询和 李军 同性别的所有同学的sname

mysql> select ssex from student where sname="李军";
+------+
| ssex |
+------+
| 男   |
+------+
1 row in set (0.00 sec)

mysql> select sname from student where ssex=(select ssex from student where sname="李军");
+-----------+
| sname     |
+-----------+
| 曾华      |
| 匡明      |
| 李军      |
| 陆军      |
| 王尼玛    |
| 张全蛋    |
| 赵铁柱    |
+-----------+
7 rows in set (0.00 sec)

# 40. 查询和 李军 同性别并且同班的所有同学的sname

mysql> select sname from student where ssex=(select ssex from student where sname="李军") and class=(select class from student where 
sname="李军");
+-----------+
| sname     |
+-----------+
| 曾华      |
| 李军      |
| 王尼玛    |
+-----------+
3 rows in set (0.00 sec)

# 42. 查询所有选修 计算机导论 课程的 男 同学的成绩表。

## 先查询男同学学号
mysql> select * from student where ssex="男";
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 101 | 曾华      | 男   | 1977-09-01 00:00:00 | 95033 |
| 102 | 匡明      | 男   | 1975-10-02 00:00:00 | 95031 |
| 104 | 李军      | 男   | 1976-02-20 00:00:00 | 95033 |
| 106 | 陆军      | 男   | 1974-06-03 00:00:00 | 95031 |
| 107 | 王尼玛    | 男   | 1976-02-20 00:00:00 | 95033 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
| 109 | 赵铁柱    | 男   | 1974-06-03 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
7 rows in set (0.00 sec)

## 查询课程号
mysql> select * from course where cname="计算机导论";
+-------+-----------------+-----+
| cno   | cname           | tno |
+-------+-----------------+-----+
| 3-105 | 计算机导论      | 825 |
+-------+-----------------+-----+
1 row in set (0.00 sec)

## 通过学号和课程号在成绩表中查询数据
mysql> select * from score where sno in (select sno from student where ssex="男") and cno =(select cno from course where cname="计算
机导论");
+-----+-------+-------+
| sno | cno   | dgree |
+-----+-------+-------+
| 101 | 3-105 |    90 |
| 102 | 3-105 |    91 |
| 104 | 3-105 |    89 |
| 109 | 3-105 |    76 |
+-----+-------+-------+
4 rows in set (0.02 sec)

```

#### 按等级查询
```mysql
# 43. 假设建立一个 grade 表代表学生的成绩等级，并插入数据

CREATE TABLE grade (
    low INT(3),
    upp INT(3),
    grade char(1)
);

INSERT INTO grade VALUES (90, 100, 'A');
INSERT INTO grade VALUES (80, 89, 'B');
INSERT INTO grade VALUES (70, 79, 'C');
INSERT INTO grade VALUES (60, 69, 'D');
INSERT INTO grade VALUES (0, 59, 'E');


# 现查询所有同学的sno、cno和grade列

mysql> select sno,cno,grade from score,grade where dgree between low and upp;
+-----+-------+-------+
| sno | cno   | grade |
+-----+-------+-------+
| 101 | 3-105 | A     |
| 102 | 3-105 | A     |
| 103 | 3-105 | A     |
| 103 | 3-245 | B     |
| 103 | 6-166 | B     |
| 104 | 3-105 | B     |
| 105 | 3-105 | B     |
| 105 | 3-245 | C     |
| 105 | 6-166 | C     |
| 109 | 3-105 | C     |
| 109 | 3-245 | D     |
| 109 | 6-166 | B     |
+-----+-------+-------+
12 rows in set (0.00 sec)
```
#### SQL 四种连接查询 
> 内连接 `inner join` 或者 `join`

> 外连接
> > 1. 左连接 `left join` 或者 `left outer join`
>> 2. 右连接 `right join` 或者 `right outer join`
>> 3. 完全外连接 `full join` 或者 `full outer join`


##### 创建两个表
> 1. person 表
> > id
> name
> cardId

> 2. card 表
> > id 
> name


```mysql
# 创建数据库 与 两个表
create database testJoin;

create table person (
    id int,
    name varchar(20),
    cardId int
);

create table card(
    id int,
    name varchar(20)
);


# 插入数据
INSERT INTO card VALUES (1, '饭卡'), (2, '建行卡'), (3, '农行卡'), (4, '工商卡'), (5, '邮政卡');
mysql> select * from card;
+------+-----------+
| id   | name      |
+------+-----------+
|    1 | 饭卡      |
|    2 | 建行卡    |
|    3 | 农行卡    |
|    4 | 工商卡    |
|    5 | 邮政卡    |
+------+-----------+
5 rows in set (0.00 sec)


INSERT INTO person VALUES (1, '张三', 1), (2, '李四', 3), (3, '王五', 6);
mysql> select * from person; 
+------+--------+--------+
| id   | name   | cardId |
+------+--------+--------+
|    1 | 张三   |      1 |
|    2 | 李四   |      3 |
|    3 | 王五   |      6 |
+------+--------+--------+
3 rows in set (0.00 sec)

```
分析两张表发现，`person` 表并没有为 `cardId` 字段设置一个在 `card` 表中对应的 `id` 外键。如果设置了的话，`person` 中 `cardId` 字段值为 `6` 的行就插不进去，因为该 `cardId` 值在 `card` 表中并没有。

##### 1. inner join 查询（内连接）
内联查询，两张表中的数据，通过某个字段相连，查询出相关的记录数据。
```mysql
# inner join 查询
mysql> select * from person inner join card on person.cardId=card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
+------+--------+--------+------+-----------+
2 rows in set (0.00 sec)

## 其中 inner 可省略
mysql> select * from person join card on person.cardId=card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
+------+--------+--------+------+-----------+
2 rows in set (0.00 sec)

```
##### 2. left join (左外连接)
左外连接，会把左边表里数据全部取出，而右表数据有相等则显示，没有就补 NULL。
```mysql
mysql> select * from person left join card on person.cardId=card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
|    3 | 王五   |      6 | NULL | NULL      |
+------+--------+--------+------+-----------+
3 rows in set (0.00 sec)

## outer 可省略
mysql> select * from person left outer join card on person.cardId=card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
|    3 | 王五   |      6 | NULL | NULL      |
+------+--------+--------+------+-----------+
3 rows in set (0.00 sec)

```
##### 3. right join (右外连接)
右外连接，会把右边表里数据全部取出，而左表数据有相等则显示，没有就补 NULL。
```mysql
mysql> select * from person right outer join card on person.cardId=card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
| NULL | NULL   |   NULL |    2 | 建行卡    |
| NULL | NULL   |   NULL |    4 | 工商卡    |
| NULL | NULL   |   NULL |    5 | 邮政卡    |
+------+--------+--------+------+-----------+
5 rows in set (0.00 sec)

## outer 可省略
mysql> select * from person right join card on person.cardId=card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
| NULL | NULL   |   NULL |    2 | 建行卡    |
| NULL | NULL   |   NULL |    4 | 工商卡    |
| NULL | NULL   |   NULL |    5 | 邮政卡    |
+------+--------+--------+------+-----------+
5 rows in set (0.00 sec)
```
##### 4. full join (全外连接)
mysql 不支持 full join
```mysql
mysql> select * from person full join card on person.cardId=card.id;
ERROR 1054 (42S22): Unknown column 'person.cardId' in 'on clause'

## 左外连接与右外连接拼接
mysql> select * from person left join card on person.cardId=card.id union select * from person right join card on person.cardId=card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
|    3 | 王五   |      6 | NULL | NULL      |
| NULL | NULL   |   NULL |    2 | 建行卡    |
| NULL | NULL   |   NULL |    4 | 工商卡    |
| NULL | NULL   |   NULL |    5 | 邮政卡    |
+------+--------+--------+------+-----------+
6 rows in set (0.00 sec)

```
## 三. mysql 事务
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