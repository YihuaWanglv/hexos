---
title: 记一次线上卡顿问题的处理-Mysql的Lock wait timeout
date: 2019-11-18 15:09:04
tags: [exception, mysql, timeout, lock]
categories: exception
---

记一次线上卡顿问题的处理-Mysql的Lock wait timeout

## 问题表现：

- 1.程序jdbc连接Mysql报错'Lock wait timeout exceeded;try restarting transaction'

- 2.页面某2个操作提交卡住，一直转

## 排查：

使用`show processlist`和`select * from information_schema.innodb_trx`会看到有一些等待中的mysql线程

可以用kill命令把这些mysql线程kill掉，但这只是暂时紧急应对，没有定位到真正的问题所在。
``
出现锁等待超时的情况，一般可能会是因为一个sql执行完，但未commit，接下来的sql要执行就会被锁，直到完成或者超时结束。

继续对问题进行分析。

出现等待的两个sql分别是`insert ignore into biz_customer_contact...`以及`update customer ...`，而这2个表只是百万级的表，正常的插入和更新表操作一般不会造成等待和锁定，因此考虑不是sql本身的问题，sql本身不用进行优化。

同时，出现锁定超时的问题仅仅是这2个地方这2个sql，而其它地方没有出现问题，因此也排除是整个数据库出现性能问题。

再深入问题，找到这2个sql执行所在的程序代码，review代码后，发现两者有一个共性特征，就是两者都需要调用一个其它系统的接口来完成业务操作。考虑到网络调用或者I/O操作是很可能出现阻塞等待的情况的，因此怀疑是接口调用等待的时间太长，导致数据库事务一直在等待，最后导致超时，出现上面的线程等待问题和异常报错。

针对这个问题，专门测试了依赖的接口和咨询对应系统的运维后，确认，依赖的系统确实当时正出现问题。因此定位到问题所在。


## 定位到问题后，就要做适当的处理，处理方案为：

- 1.恢复出现问题的服务
- 2.将接口调用的超时时间缩短，原来是没有设置这个时间的。缩短接口超时时间的目的是假如服务不可用，则快速失败抛出错误，避免系统一直等待卡顿。
- 3.适当设置mysql数据库的锁等待超时时间，例如`set GLOBAL innodb_lock_wait_timeout = 5000`，缩短等待超时时间，目的也是避免高并发时太多线程在等待拖垮系统。