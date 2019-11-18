---
title: Mysql的PARTITION表分区要点
date: 2019-11-18 11:46:18
tags: [mysql, PARTITION, 表分区, 分表]
categories: mysql
---



Mysql自带的Partition分区方案好处：

优势1，数据库操作变更简单；
优势2，分区后程序不需要修改，数据库分区的表使用和分区前一致；
优势3，分区后，查询效率没有降低。



# 1. 建立分区步骤

现在假设`biz_process`和`biz_follow`是2张大表，现在要对它们进行mysql数据库表分区


有2种推荐的方式进行表分区：

- 一是，先进行备份，再对需要分区的表直接使用`ALTER TABLE`进行分区设置

- 二是，先创建使用别名的分区表，再复制数据进分区表，最后变更表名字原来的表和分区表切换。



## 1.1 创建需要分区的表的备份

```sql
CREATE TABLE `biz_process_bak` (
  `process_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '流程编号',
  `customer_id` bigint(20) NOT NULL COMMENT '客户编号',
  `dept_id` bigint(20) NOT NULL COMMENT '业务部门编号',
  `user_id` bigint(20) DEFAULT NULL COMMENT '业务人员编号',
  --···建表语句
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='商机流程表';

CREATE TABLE `biz_follow_bak` (
  `follow_id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '跟进编号',
  `customer_id` bigint(11) NOT NULL COMMENT '客户编号',
  `process_id` bigint(11) NOT NULL COMMENT '流程编号',
  --···建表语句
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='客户跟进表';

insert into biz_process_bak select * from biz_process;
insert into biz_follow_bak select * from biz_follow;



```

## 1.2 创建别名分区表，并复制数据进分区表

```sql
CREATE TABLE `biz_process_range` (
  `process_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '流程编号',
  `customer_id` bigint(20) NOT NULL COMMENT '客户编号',
  `dept_id` bigint(20) NOT NULL COMMENT '业务部门编号',
  `user_id` bigint(20) DEFAULT NULL COMMENT '业务人员编号',
  --···建表语句
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='商机流程表' PARTITION BY RANGE(process_id) (
    PARTITION p1 VALUES LESS THAN (2000000),
    PARTITION p2 VALUES LESS THAN (4000000),
    PARTITION p3 VALUES LESS THAN (6000000),
    PARTITION p4 VALUES LESS THAN (8000000),
    PARTITION p5 VALUES LESS THAN (10000000),
    PARTITION p6 VALUES LESS THAN (12000000),
    PARTITION p7 VALUES LESS THAN (14000000),
    PARTITION p8 VALUES LESS THAN (16000000),
    PARTITION p9 VALUES LESS THAN (18000000),
    PARTITION p10 VALUES LESS THAN (20000000),
    PARTITION p11 VALUES LESS THAN (MAXVALUE) 
);

CREATE TABLE `biz_follow_range` (
  `follow_id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '跟进编号',
  `customer_id` bigint(11) NOT NULL COMMENT '客户编号',
  `process_id` bigint(11) NOT NULL COMMENT '流程编号',
  --···建表语句
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='客户跟进表' PARTITION BY RANGE(follow_id) (
    PARTITION p1 VALUES LESS THAN (2000000),
    PARTITION p2 VALUES LESS THAN (4000000),
    PARTITION p3 VALUES LESS THAN (6000000),
    PARTITION p4 VALUES LESS THAN (8000000),
    PARTITION p5 VALUES LESS THAN (10000000),
    PARTITION p6 VALUES LESS THAN (12000000),
    PARTITION p7 VALUES LESS THAN (14000000),
    PARTITION p8 VALUES LESS THAN (16000000),
    PARTITION p9 VALUES LESS THAN (18000000),
    PARTITION p10 VALUES LESS THAN (20000000),
    PARTITION p11 VALUES LESS THAN (MAXVALUE) 
);

insert into biz_process_range select * from biz_process;
insert into biz_follow_range select * from biz_follow;
```

## 1.3 使用重命名快速把原表切换到分区表

```sql
RENAME TABLE biz_process TO biz_process_tmp, biz_process_range TO biz_process;
RENAME TABLE biz_follow TO biz_follow_tmp, biz_follow_range TO biz_follow;
```

ps. 也可以直接使用ALTER TABLE的方式，直接为原来的表添加分区



# 2. 关于分区新增和变更删除等操作

## 2.1 为当前表创建分区

因为是对已有表进行改造，所以只能用 alter 的方式：
```sql
ALTER TABLE stat
    PARTITION BY RANGE(TO_DAYS(dt)) (
        PARTITION p0 VALUES LESS THAN(0),
        PARTITION p190214 VALUES LESS THAN(TO_DAYS('2019-02-14')),
        PARTITION pm VALUES LESS THAN(MAXVALUE)
    );
```

复制代码这里有2点要注意：

- 一是 p0 分区，这是因为 MySQL（我是5.7版） 有个 bug，就是不管你查的数据在哪个区，它都会扫一下第一个区，我们每个区的数据都有几十万条，扫一下很是肉疼啊，所以为了避免不必要的扫描，直接弄个0数据分区就行了。
- 二是 pm 分区，这个是最大分区。假如不要 pm，那你存 2019-02-15 的数据就会报错。所以 pm 实际上是给未来的数据一个预留的分区。


## 2.2 定期扩展分区

由于 MySQL 的分区并不能自己动态扩容，所以我们要写个代码为它动态的增加分区。

增加分区需要用到 REORGANIZE 命令，它的作用是对某个分区重新分配。

比如明天是 15 号，那我们要给 15 号也增加个分区，实际上就是把 pm 分区拆分成2个分区：
```sql
ALTER TABLE stat
    REORGANIZE PARTITION pm INTO (
        PARTITION p190215 VALUES LESS THAN(TO_DAYS('2019-02-15')),
        PARTITION pm VALUES LESS THAN(MAXVALUE)
    );
```
复制代码这里就涉及到一个问题，即如何获得当前表的所有分区？网上有挺多方法，但我试了下感觉还是先 `show create table stat `然后用正则匹配出所有分区更方便一点。


## 2.3 定期删除分区

随着数据库越来越大，我们肯定是要清除旧的数据，同时也要清除旧的分区。
这个也比较简单：
```sql
ALTER TABLE stat DROP PARTITION p190214, p190215
```


# 3. 附录：Mysql的Partition分区使用说明

## 3.1 基于时间进行range分区例子：
```sql
CREATE TABLE my_range_datetime(
    id INT,
    hiredate DATETIME
) ENGINE=INNODB PARTITION BY RANGE (YEAR(hiredate) ) (
    PARTITION p1 VALUES LESS THAN (2011),
    PARTITION p2 VALUES LESS THAN (2012),
    PARTITION p3 VALUES LESS THAN (2013),
    PARTITION p4 VALUES LESS THAN (2014),
    PARTITION p5 VALUES LESS THAN (2015),
    PARTITION p6 VALUES LESS THAN (2016),
    PARTITION p7 VALUES LESS THAN (2017),
    PARTITION p8 VALUES LESS THAN (2018),
    PARTITION p9 VALUES LESS THAN (2019),
    PARTITION p10 VALUES LESS THAN (2020),
    PARTITION p11 VALUES LESS THAN (MAXVALUE) 
);
```

## 3.2 基于主键进行range分区例子：由于表如果设置了主键的话，分区的字段一定要属于主键，所以一般要么把分区字段添加为主键，要么就只能针对主键进行分区
```sql
CREATE TABLE `range_process` (
  `process_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '流程编号',
  `customer_id` bigint(20) NOT NULL COMMENT '客户编号',
  `dept_id` bigint(20) NOT NULL COMMENT '业务部门编号',
  `memo` varchar(100) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '备注',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`process_id`),
  KEY `IX_DEPT_ID` (`dept_id`),
  KEY `IX_CREATE_TIME` (`create_time`),
  KEY `IX_CUSTOMER_ID` (`customer_id`) USING BTREE,
  KEY `IX_MEMO` (`memo`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='商机流程表' PARTITION BY RANGE(process_id) (
    PARTITION p1 VALUES LESS THAN (1000),
    PARTITION p2 VALUES LESS THAN (2000),
    PARTITION p3 VALUES LESS THAN (3000),
    PARTITION p4 VALUES LESS THAN (MAXVALUE) 
);
```

## 3.3 分区后，如果在查询时想要避免搜索所有分区，则最好在查询的where条件中添加分区字段作为过滤条件。哪怕是冗余的没有必要的过滤条件，对于定位分区还是有用的。

例如，普通查询
```
mysql> EXPLAIN PARTITIONS select * from range_process \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: range_process
   partitions: p1,p2,p3,p4
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 20
     filtered: 100.00
        Extra: NULL
1 row in set, 2 warnings (0.00 sec)
```

添加主键作为where条件
```
EXPLAIN PARTITIONS select * from range_process where process_id > 3000 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: range_process
   partitions: p4
         type: range
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 8
          ref: NULL
         rows: 10
     filtered: 100.00
        Extra: Using where
1 row in set, 2 warnings (0.00 sec)
```

- 在应用层，对于做了分区的大表，进行查询时，可以通过动态sql的方式，在需要优化的sql查询处加入分区字段作为where的过滤条件。


## 3.4 分区表使用的限制

- 一个表最多只能有1024个分区
- 主键和唯一索引的列，必须作为分区的字段

## 3.5 分区后，一般情况下，各种查询使用和join联表操作均正常使用，和分区前一致，但要注意几个问题

- 关于优化，应该尽量使用where条件，将检索的分区限制再少数分区中。
- （mysql5.5以上不需要考虑这个问题）NULL值会使分区过滤无效，并在分区中创建一个默认的第一个分区，从而导致所有查询都会默认去检索这个分区，所以，应该创建一个“无用”的第一个分区，导致数据无法落入这个分区，从而避免mysql查询每次检索第一个分区。mysql5.5以上不需要考虑这个问题，可以直接使用列本身进行分区，就不会有这个问题。
- 对于大多数系统，100个左右的分区是可以接受的，不会有太大问题。







