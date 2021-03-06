---
title: java后端项目开发规范
date: 2019-11-04 15:49:28
tags: [java,后端,开发规范]
categories: manager
---






# 一. mysql数据库

## （一）建表约定

### 1.【强制】表名、字段名必须使用小写字母或数字，禁止出现数字开头，禁止两个下划线中间只出现数字。数据库字段名的修改代价很大，因为无法进行预发布，所以字段名称需要慎重考虑。

说明：MySQL 在 Windows 下不区分大小写，但在 Linux下默认是区分大小写。因此，数据库名、表名、字段名，都不允许出现任何大写字母，避免节外生枝。

正例：aliyun_admin，rdc_config，level3_name

反例：AliyunAdmin，rdcConfig，level_3_name

### 2.【强制】小数类型为 decimal，在需要精确经度的时候禁止使用 float和double。

说明：float 和 double在存储的时候，存在精度损失的问题，很可能在值的比较时，得到不正确的结果。如果存储的数据范围超过decimal的范围，建议将数据拆成整数和小数分开存储。

### 3.【强制】表示状态类型等的数字可枚举类的字段，数据类型应该使用tinyint。如果字段为非负数，数据类型是unsigned tinyint.

说明：任何字段如果为非负数，必须是 unsigned。

### 4.【强制】表名不使用复数名词

### 5.【强制】主键索引名为 pk_字段名；唯一索引名为 uk_字段名；普通索引名则为 idx_字段名。

说明：pk_ 即 primary key；uk_ 即 unique key；idx_ 即 index 的简称。

### 6.【强制】如果存储的字符串长度几乎相等，使用 char定长字符串类型。

### 7.【强制】varchar是可变长字符串，不预先分配存储空间，长度不要超过 5000，如果存储长度大于此值，定义字段类型为 text，独立出来一张表，用主键来对应，避免影响其它字段索引效率。

### 8.【推荐】表的命名最好是加上“业务名称_表的作用”。

### 9.【推荐】字段允许适当冗余，以提高查询性能，但必须考虑数据一致。
冗余字段应遵循：

&emsp;1）不是频繁修改的字段。

&emsp;2）不是 varchar 超长字段，更不能是 text 字段。

正例：商品类目名称使用频率高，字段长度短，名称基本一成不变，可在相关联的表中冗余存储类目名称，避免关联查询。


## （二）索引约定

### 1.【强制】超过三个表禁止 join。需要join的字段，数据类型必须绝对一致；多表关联查询时，保证被关联的字段需要有索引。

说明：即使双表 join 也要注意表索引、SQL 性能。

### 2.【强制】在varchar字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据实际文本区分度决定索引长度即可。
说明：索引的长度与区分度是一对矛盾体，一般对字符串类型数据，长度为 20 的索引，区分度会高达 90%以上，可以使用 count(distinct left(列名, 索引长度))/count(*)的区分度
来确定。

### 3.【推荐】页面搜索严禁左模糊或者全模糊，如果需要请走搜索引擎来解决。
说明：索引文件具有B-Tree的最左前缀匹配特性，如果左边的值未确定，那么无法使用此索
引。

### 4.【推荐】如果有 order by的场景，请注意利用索引的有序性。order by最后的字段是组合索引的一部分，并且放在索引组合顺序的最后，避免出现 file_sort 的情况，影响查询性能。

正例：where a=? and b=? order by c; 索引：a_b_c

反例：索引中有范围查找，那么索引有序性无法利用，如：WHERE a>10 ORDER BY b; 索引
a_b 无法排序。

### 5.【推荐】利用延迟关联或者子查询优化超多分页场景。

说明：MySQL 并不是跳过 offset 行，而是取 offset+N 行，然后返回放弃前 offset 行，返回N行，那当offset特别大的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特定阈值的页数进行 SQL 改写。

正例：先快速定位需要获取的 id 段，然后再关联：
```
 SELECT a.* FROM 表 1 a, (select id from 表 1 where 条件 LIMIT 100000,20 ) b where a.id=b.id
```

### 6.【推荐】建组合索引的时候，区分度最高的在最左边。

正例：如果
```
where a=? and b=?
```
，a列的几乎接近于唯一值，那么只需要单建idx_a索引即可。

说明：存在非等号和等号混合判断条件时，在建索引时，请把等号条件的列前置。如：
```
where a>?and b=?
```
那么即使 a 的区分度更高，也必须把 b 放在索引的最前列。


## (三)SQL语句约定

### 1.【强制】不得使用外键与级联，一切外键概念必须在应用层解决。

说明：以学生和成绩的关系为例，学生表中的student_id是主键，那么成绩表中的student_id则为外键。如果更新学生表中的 student_id，同时触发成绩表中的 student_id 更新，即为级联更新。外键与级联更新适用于单机低并发，不适合分布式、高并发集群；级联更新是强阻塞，存在数据库更新风暴的风险；外键影响数据库的插入速度。


### 2.【强制】不要使用 count(列名)或 count(常量)来替代 count(*)，count(*)是 SQL92 定义的标准统计行数的语法，跟数据库无关，跟 NULL 和非 NULL 无关。

说明：count(*)会统计值为 NULL 的行，而 count(列名)不会统计此列为 NULL 值的行。

### 3.【强制】当某一列的值全是 NULL 时，count(col)的返回结果为 0，但 sum(col)的返回结果为NULL，因此使用 sum()时需注意 NPE 问题。

正例：可以使用如下方式来避免 sum 的 NPE 问题：
```
SELECT IF(ISNULL(SUM(g)),0,SUM(g)) FROM table;
```

### 4.【强制】在代码中写分页查询逻辑时，若 count 为 0 应直接返回，避免执行后面的分页语句。

### 5.【强制】禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。

### 6.【强制】数据订正时，删除和修改记录时，要先 select，避免出现误删除，确认无误才能执
行更新语句。

### 7.【推荐】in 操作能避免则避免，若实在避免不了，需要仔细评估 in 后边的集合元素数量，控
制在 1000 个之内。

### 8.【参考】如果有全球化需要，所有的字符存储与表示，均以 utf-8 编码，注意字符统计函数的区别。

说明：

- SELECT LENGTH("轻松工作")； 返回为 12
- SELECT CHARACTER_LENGTH("轻松工作")； 返回为 4

如果需要存储表情，那么选择 utfmb4 来进行存储，注意它与 utf-8 编码的区别。



## (四)ORM 映射 

### 1.【强制】在表查询中，一律不要使用 *作为查询的字段列表，需要哪些字段必须明确写明。

说明：

    1）增加查询分析器解析成本。
    2）增减字段容易与 resultMap 配置不一致。

### 2.【强制】不要用 resultClass 当返回参数，即使所有类属性名与数据库字段一一对应，也需要定义；反过来，每一个表也必然有一个与之对应。

说明：配置映射关系，使字段与 DO 类解耦，方便维护。

### 3.【强制】sql.xml 配置参数使用：#{}，#param# 不要使用${} 此种方式容易出现 SQL 注入。

### 4.【推荐】不要直接拿 HashMap 与 Hashtable 作为查询结果集的输出。

说明：resultClass=”Hashtable”，会置入字段名和属性值，但是值的类型不可控。

### 5.【推荐】不要写一个大而全的数据更新接口。传入为 POJO类，不管是不是自己的目标更新字段，都进行 update table set c1=value1,c2=value2,c3=value3; 这是不对的。执行 SQL时，不要更新无改动的字段，一是易出错；二是效率低；三是增加binlog存储.




# 2. 工程结构

## (一)应用分层

### 1. 【推荐】图中默认上层依赖于下层，箭头关系表示可直接依赖，如：开放接口层可以依赖于Web 层，也可以直接依赖于 Service 层，依此类推：

![](images/java-file-001.png)

- __开放接口层__：可直接封装 Service 方法暴露成 RPC 接口；通过 Web 封装成 http 接口；进行网关安全控制、流量控制等。
- __终端显示层__：各个端的模板渲染并执行显示的层。当前主要是 velocity 渲染，JS 渲染，JSP 渲染，移动端展示等。
- __Web层__：主要是对访问控制进行转发，各类基本参数校验，或者不复用的业务简单处理等。
- __Service 层__：相对具体的业务逻辑服务层。
- __Manager 层__：通用业务处理层，它有如下特征：

    &emsp;1） 对第三方平台封装的层，预处理返回结果及转化异常信息；

    &emsp;2） 对 Service 层通用能力的下沉，如缓存方案、中间件通用处理；

    &emsp;3） 与 DAO 层交互，对多个 DAO 的组合复用。
- __DAO 层__：数据访问层，与底层 MySQL、Oracle、Hbase 等进行数据交互。
 外部接口或第三方平台：包括其它部门 RPC 开放接口，基础平台，其它公司的 HTTP 接口。



## (二)二方库依赖

### 1. 【强制】定义 GAV 遵从以下规则：

&emsp;1） GroupID 格式：com.{公司/BU }.业务线.[子业务线]，最多 4 级。
说明：{公司/BU} 例如：alibaba/taobao/tmall/aliexpress 等 BU 一级；子业务线可选。
正例：com.taobao.jstorm 或 com.alibaba.dubbo.register

&emsp;2） ArtifactID 格式：产品线名-模块名。语义不重复不遗漏，先到中央仓库去查证一下。
正例：dubbo-client / fastjson-api / jstorm-tool

&emsp;3） Version：详细规定参考下方。

### 2. 【强制】二方库版本号命名方式：主版本号.次版本号.修订号

&emsp;1） 主版本号：产品方向改变，或者大规模 API 不兼容，或者架构不兼容升级。

&emsp;2） 次版本号：保持相对兼容性，增加主要功能特性，影响范围极小的 API 不兼容修改。

&emsp;3） 修订号：保持完全兼容性，修复 BUG、新增次要功能特性等。

说明：注意起始版本号必须为：1.0.0，而不是 0.0.1正式发布的类库必须先去中央仓库进
行查证，使版本号有延续性，正式版本号不允许覆盖升级。如当前版本：1.3.3，那么下一个合理的版本号：1.3.4 或 1.4.0 或 2.0.0

### 3. 【强制】线上应用不要依赖 SNAPSHOT 版本（安全包除外）。

说明：不依赖SNAPSHOT版本是保证应用发布的幂等性。另外，也可以加快编译时的打包构建。


# 3. 编程约定