# MySQL 5.7 表分区技术(二)：RANGE分区



# 一、说明

- 文档示例采用的MySQL版本为`5.7.21-log`，不同版本可能会有些区别。

- 示例表基本结构如下：

  ```mysql
  DROP TABLE IF EXISTS `user`;
  CREATE TABLE `user` (
    `id` int(11) NOT NULL COMMENT '用户id',
    `login_name` varchar(20) NOT NULL COMMENT '登录名',
    `user_name` varchar(20) NOT NULL COMMENT '用户名',
    `join_date` date NOT NULL COMMENT '加入日期',
    `position_code` int(11) NOT NULL COMMENT '岗位代码',
    `dept_id` int(11) NOT NULL COMMENT '部门ID'
  ) COMMENT='用户表';
  ```

- 为了方便，创建的示例表不包含**主键**和**唯一索引**，因为分区对主键和唯一索引有点特殊要求，后续将用专门章节介绍。



# 二、RANGE分区定义 

`RANGE`分区，就是每个分区都包含分区表达式值位于给定范围内的数据，也就是说分区表达式的值位于哪个给定范围内就将数据放到哪个分区里。范围应该是连续的且不能重叠，并使用`VALUES LESS THAN`运算符定义。类似于JAVA语言里的`if...else if...else if...`。

RANGE分区的SQL语法，是在`create table`语句后增加`partition by range`子句。

```mysql
create table table_name(
  column0,
  column1,
  ...
)
partition by range(用来分区的列名)(
	partition 分区0名称 values less than(对应条件值),
  partition 分区1名称 values less than(对应条件值),
  partition 分区2名称 values less than(对应条件值),
  ...
);
```

需要注意的是，每个分区的范围不能重复，并且都是按顺序定义的，从低到高。



# 三、示例

## 3.1 根据dept_id列进行分区

将部门id小于11的一个分区，大于等于11小于21的一个分区，大于等于21小于31的一个分区。

```mysql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL COMMENT '用户id',
  `login_name` varchar(20) NOT NULL COMMENT '登录名',
  `user_name` varchar(20) NOT NULL COMMENT '用户名',
  `join_date` date NOT NULL COMMENT '加入日期',
  `position_code` int(11) NOT NULL COMMENT '岗位代码',
  `dept_id` int(11) NOT NULL COMMENT '部门ID'
) COMMENT='用户表'
partition by range (dept_id)(
    partition part0 values less than (11),
    partition part1 values less than (21),
    partition part2 values less than (31)
);
```

通过`show create table user\G`查看表结构定义，如下：

```mysql
mysql> show create table user\G;
*************************** 1. row ***************************
       Table: user
Create Table: CREATE TABLE `user` (
  `id` int(11) NOT NULL COMMENT '用户id',
  `login_name` varchar(20) NOT NULL COMMENT '登录名',
  `user_name` varchar(20) NOT NULL COMMENT '用户名',
  `join_date` date NOT NULL COMMENT '加入日期',
  `position_code` int(11) NOT NULL COMMENT '岗位代码',
  `dept_id` int(11) NOT NULL COMMENT '部门ID'
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户表'
/*!50100 PARTITION BY RANGE (dept_id)
(PARTITION part0 VALUES LESS THAN (11) ENGINE = InnoDB,
 PARTITION part1 VALUES LESS THAN (21) ENGINE = InnoDB,
 PARTITION part2 VALUES LESS THAN (31) ENGINE = InnoDB) */
1 row in set (0.00 sec)
```

可以看到表结构定义里包含了分区的定义，这里`/*!50100  ...   */`是一种特殊的注释，它表示符合条件时执行，`50100`表示`5.01.00`版本，所以这里注释的意思是表示`5.01.00`版本及更高版本时才执行。

这种分区方式，插入部门id为1~10时会存放在part0分区里，插入部门id为11~20时存放在part1分区里，以此类推。下面插入4条数据到user表。

```mysql
INSERT INTO user VALUES (1, 'user1', '用户1', '2021-10-01', 1, 1);
INSERT INTO user VALUES (2, 'user2', '用户2', '2021-11-01', 1, 5);
INSERT INTO user VALUES (3, 'user3', '用户3', '2021-12-01', 1, 14);
INSERT INTO user VALUES (4, 'user4', '用户4', '2021-12-05', 1, 22);
```

插入到分区表和插入到普通的表一样，即用户读写分区表时和读写普通表没什么两样。分别执行下面4个sql查看数据，可以看出我们插入的数据已按规则存到了相应的分区。

```mysql
select * from user;  /* 查询整个表，可以查到4条数据 */

select * from user partition (part0);   /* 只查询表的part0分区，查到2条数据，user1和user2 */

select * from user partition (part1);		/* 只查询表的part1分区，查到1条数据，user3 */

select * from user partition (part2);		/* 只查询表的part2分区，查到1条数据，user4 */
```

现在如果我们插入一个部门ID为31的用户，会出现什么情况呢？

```mysql
INSERT INTO user VALUES (31, 'user31', '用户31', '2021-12-01', 1, 31);
```

执行上面的insert语句，提示`[HY000][1526] Table has no partition for value 31`，错误信息很明确，就是说我们部门id为31的这条数据插入时，根据分区定义规则，未找到对应的分区，MySQL服务器不知道要放哪个分区里。因为不确定公司将来发展成多大，会有多少部门，那么有没有办法让大于等于31的所有数据都放一个分区呢？通过`MAXVALUE`关键字可以实现这种效果，我继续看下一小节。

## 3.2 通过MAXVALUE关键字"catch all"所有大于某个值的数据

`MAXVALUE`表示一个整数值，它总是大于可能的最大整数值，所有部门id大于等31的都会存入该分区，表创建如下：

```mysql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL COMMENT '用户id',
  `login_name` varchar(20) NOT NULL COMMENT '登录名',
  `user_name` varchar(20) NOT NULL COMMENT '用户名',
  `join_date` date NOT NULL COMMENT '加入日期',
  `position_code` int(11) NOT NULL COMMENT '岗位代码',
  `dept_id` int(11) NOT NULL COMMENT '部门ID'
) COMMENT='用户表'
partition by range (dept_id)(
    partition part0 values less than (11),
    partition part1 values less than (21),
    partition part2 values less than (31),
    partition part3 values less than maxvalue
);
```

我们只是把上小节的创建表语句增加了一个分区part3，并且part3对应的值是`MAXVALUE`。通过show查看表结构如下：

```mysql
mysql> show create table user\G;
*************************** 1. row ***************************
       Table: user
Create Table: CREATE TABLE `user` (
  `id` int(11) NOT NULL COMMENT '用户id',
  `login_name` varchar(20) NOT NULL COMMENT '登录名',
  `user_name` varchar(20) NOT NULL COMMENT '用户名',
  `join_date` date NOT NULL COMMENT '加入日期',
  `position_code` int(11) NOT NULL COMMENT '岗位代码',
  `dept_id` int(11) NOT NULL COMMENT '部门ID'
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户表'
/*!50100 PARTITION BY RANGE (dept_id)
(PARTITION part0 VALUES LESS THAN (11) ENGINE = InnoDB,
 PARTITION part1 VALUES LESS THAN (21) ENGINE = InnoDB,
 PARTITION part2 VALUES LESS THAN (31) ENGINE = InnoDB,
 PARTITION part3 VALUES LESS THAN MAXVALUE ENGINE = InnoDB) */
1 row in set (0.01 sec)
```

现在我们再执行刚才的部门id为31的sql语句进行插入，就不报错了，会将数据存入到part3分区里。

```mysql
INSERT INTO user VALUES (31, 'user31', '用户31', '2021-12-01', 1, 31);
```

假如将来公司发展的很大，部门增加的很多，想把part3分区里面的数据分开，那么可以通过`alter table`语句修改添加新的分区，这个将在后面的文章里讲解说明，这里不再细说。

除了增加分区的方式，还有另外一种处理因没有匹配到分区而报错的方法，也就是通过使用**ignore**关键字忽略此类错误，如果这样，在插入时直接丢弃没有匹配到分区的数据，则不会插入包含不匹配的分区列值的行，只会插入能匹配的任何行，并且不会报任何错误。

```mysql
INSERT IGNORE INTO user VALUES (31, 'user31', '用户31', '2021-12-01', 1, 31);
```





## 3.3 根据position_code对表分区

前面我们是根据部门id来分区的，根据岗位代码分区也一样，比如，把像我这种敲代码的一线员工放到一个分区(position_code<100)，将项目经理、产品经理等放到1个分区(100<=position_code<200)，将部门经理及以上的大领导放入一个分区(position_code>=200)。

```mysql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL COMMENT '用户id',
  `login_name` varchar(20) NOT NULL COMMENT '登录名',
  `user_name` varchar(20) NOT NULL COMMENT '用户名',
  `join_date` date NOT NULL COMMENT '加入日期',
  `position_code` int(11) NOT NULL COMMENT '岗位代码',
  `dept_id` int(11) NOT NULL COMMENT '部门ID'
) COMMENT='用户表'
partition by range (position_code)(
    partition part0 values less than (100),
    partition part1 values less than (200),
    partition part2 values less than maxvalue
);
```

岗位代码小于100的用户将放入part0分区，大于等100小于200的用户放入part1分区，所有大于等于200的用户放入part2分区。因为和根据dept_id分区一样，所以这里就不再写测试脚本了。



## 3.4 子句使用表达式

前面的例子都是通过指定类型为int的列的列名，MySQL也可以通过指定表达式的方式，但是，表达式必须能被MySQL服务器解析计算并返回一个int值，使能够和`LESS TAHN`指定的值进行比较。

比如我们可以根据基于类型为`date`的`join_date`列的表达式进行分区，比如`year(join_date)`表达式。我们将2018年以前入职的放入一个分区，2019年入职的放入一个分区，2020年入职的放入一个分区，2020年以后入职的都放入一个分区。创建方式如下：

```mysql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL COMMENT '用户id',
  `login_name` varchar(20) NOT NULL COMMENT '登录名',
  `user_name` varchar(20) NOT NULL COMMENT '用户名',
  `join_date` date NOT NULL COMMENT '加入日期',
  `position_code` int(11) NOT NULL COMMENT '岗位代码',
  `dept_id` int(11) NOT NULL COMMENT '部门ID'
) COMMENT='用户表'
partition by range (year(join_date))(
    partition part0 values less than (2018),
    partition part1 values less than (2019),
    partition part2 values less than (2020),
    partition part3 values less than maxvalue
);
```

分区表达式除了支持这里使用的`year()`函数，还支持一些其他行数，具体参考下一节介绍。



# 四、分区表达式支持的函数

分区表达式中，并不是能使用所有的函数，只支持以下函数：

- `ABS()`
- `CEILING()`
- `DAY()`
- `DAYOFMONTH()`
- `DAYOFWEEK()`
- `DAYOFYEAR()`
- `DATEDIFF()`
- `EXTRACT()`
- `FLOOR()`
- `HOUR()`
- `MICROSECOND()`
- `MINUTE()`
- `MOD()`
- `QUARTER()`
- `SECOND()`
- `TIME_TO_SEC()`
- `TO_DAYS()`
- `TO_SECONDS()`
- `UNIX_TIMESTAMP()`,(使用`TIMESTAMP`类型的列时)
- `WEEKDAY()`
- `YEAR()`
- `YEARWEEK()`

需要注意的是，`CEILING()`和`FLOOR()`函数，只支持返回值精确数组的情况(如`INT`或`DECIMAL`)，返回浮点类型的数值时不支持(如`FLOAT`)。如下创建脚本将报错：

```mysql
create table table_name(
    num float
)
partition by range (floor(num))(
    partition p0 values less than (10),
    partition p1 values less than (20),
    partition p2 values less than maxvalue
);
```

执行将会报`[HY000][1491] The PARTITION function returns the wrong type`错误。但以下脚本将可以正常执行：

```mysql
create table table_name(
    num int
)
partition by range (floor(num))(
    partition p0 values less than (10),
    partition p1 values less than (20),
    partition p2 values less than maxvalue
)
```

这里只是将num字段由float改为了int。也就是说floor()函数返回整型数据时才可以作为分区列。



另外`UNIX_TIMESTAMP()`是需要作用于timestamp类型的类上才可以。以下脚本将报错：

```mysql
DROP TABLE IF EXISTS `table_name`;
create table table_name(
    update_time datetime
)
partition by range (unix_timestamp(update_time))(
    partition p0 values less than (unix_timestamp('2021-10-01 00:00:00')),
    partition p1 values less than (unix_timestamp('2021-11-01 00:00:00')),
    partition p2 values less than (unix_timestamp('2021-12-01 00:00:00')),
    partition p3 values less than maxvalue
);
```

执行将报`[HY000][1486] Constant, random or timezone-dependent expressions in (sub)partitioning function are not allowed`错误。将update_time列类型由datetime类型改为timestamp类型，再次执行将可以正常执行，如下：

```mysql
DROP TABLE IF EXISTS `table_name`;
create table table_name(
    update_time timestamp
)
partition by range (unix_timestamp(update_time))(
    partition p0 values less than (unix_timestamp('2021-10-01 00:00:00')),
    partition p1 values less than (unix_timestamp('2021-11-01 00:00:00')),
    partition p2 values less than (unix_timestamp('2021-12-01 00:00:00')),
    partition p3 values less than maxvalue
);
```



# 五、RANGE分区适用的情况

当有以下情况时，范围分区特别有用：

- 当想要或需要删除旧数据时。通过删除旧数据所在分区(`ALTER TABLE employees DROP PARTITION p0;`)，可以很简单地删除数据。对于包含大量行的表，这比通过delete方式方便的多`DELETE FROM employees WHERE YEAR(separated) <= 1990;`，分区最长久的用途之一就是按日期进行分区隔离数据。

- 经常执行直接依赖于对表进行分区的列的查询。
- 当想要使用包含日期、时间等的列时。












