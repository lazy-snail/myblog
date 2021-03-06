---
title: MySQL
date: 2018-04-09 17:03:14
categories: DB
tags: [DB, MySQL]
---
[toc]
单进程多线程数据库模型。


# 相关概念

## MVCC
多版本并发控制。包括 MySQL、Oracle、PostgreSQL 等数据库系统都实现了MVCC，尽管实现机制不尽相同。
可以认为 MVCC 是行级锁的一个变种，在很多情况下避免了加锁操作，从而减小开销。实现方式一般为 **读操作为非阻塞式的，写操作只锁定必要的行**。
理想的 MVCC 能够解决幻读问题，但难以实现，因为企图通过乐观锁代替两阶段提交。修改两行数据，但为了保证其一致性，与修改两个分布式系统中的数据并无区别，而两阶段提交是目前这种场景保证一致性的唯一手段。**两阶段提交的本质是锁定，乐观锁的本质是消除锁定**，二者矛盾，故理想的 MVCC 难以真正在实际中被应用。包括 InnoDB 也只是借了 MVCC 这个名字，提供了非阻塞读，但真正解决幻读问题使用的是 Next-Key Lock。

[参考]
https://blog.csdn.net/chen77716/article/details/6742128
