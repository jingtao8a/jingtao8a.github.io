---
title: 13.redo日志
date: 2023-05-27 14:22:32
tags: [MySQL,MySQL是怎样运行的]
categories: 数据库
---

与在事务提交时将所有修改过的内存中的页面刷新到磁盘中相比，只将该事务执行过程中产生的redo日志刷新到磁盘的好处如下
1.redo日志占用的空间非常小
2.redo日志是顺序写入磁盘的

每条语句包含多个mtr，每个mtr包含一组redo log
一个mtr运行结束后，会将产生的一组redolog复制到log buffer中，在一些情况下它们会被刷新到磁盘里
1.log buffer空间不足时
2.事务提交时
3.后台线程不停地刷
4.正常关闭服务器
5.做checkpoint 时
6.其它情况

redo日志文件前4个block
- log file header：描述该日志文件地一些整体属性
- checkpoint1
- 无用
- checkpoint2

Log Sequence Number（日志序列号）lsn
每一组由mtr生成地redo日志都有一个唯一的lsn值与其对应，lsn值越小，说明redo日志产生的越早


在mtr结束时，还会将执行过程中可能修改过的页面加入到buffer pool 的flush链表

checkpoint：
redo日志只是为了系统崩溃后恢复脏页用的，如果对应的脏页已经刷新到磁盘，就不需要对应的redo日志了，所以判断某些redo日志占用的磁盘空间是否可以覆盖的依据就是它对应的脏页是否已经刷新到磁盘里。

做一次checkpoint其实可以分为两个步骤
1.计算一下当前系统中可以被覆盖的redo日志对应的lsn值最大是多少（有必要的话更新checkpoint_lsn）
2.将checkpoint_lsn和对应的redo日志文件组偏移量以及此次checkpoint的编号写到日志文件的管理信息（目前系统做了多少次checkpoint的变量checkpoint_no，每做一次checkpoint，该变量就加1）