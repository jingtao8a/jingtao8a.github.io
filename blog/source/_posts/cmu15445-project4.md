---
title: cmu15445-project4
date: 2024-02-29 04:11:36
tags: cmu15445—2023
categories: cmu15445-2023
---

## Concurrency Control Theory
### formal Definitions
Database: A fixed set of named data objects (e.g., A, B, C, ...)
Transaction: A sequence of read and write operations (e.g., R(A), W(B), ...)

transaction的正确性标准ACID：
- Atomicity: 原子性 "all or nothing"
- Consistency: 一致性 "it looks correct to me"
- Isolation: 隔离性 "as if alone"
- Durability: 持久性 "survive failures"

### Conflicting Operations
多个transaction并发执行时会发生冲突:
- 读写冲突(R-W)
- 写读冲突(W-R)
- 写写冲突(W-W)

读写冲突造成的问题：<br>
- 不可重复读(在一个事务内多次读取同一个数据，如果出现前后两次读到的数据不一样的情况，就意味着发生了[不可重复读]的现象)
- 幻读：在一个事务内多次查询某个符合查询条件的[记录数量]，出现前后两次查询到的记录数量不一样

写读冲突造成的问题：<br>
- 脏读(一个事务[读到]了另一个[未提交事务修改过的数据],就意味着发生了[脏读]现象)

写写冲突造成的问题: <br>
- 丢失修改

![img](../images/cmu15445-project4/2.png)
![img](../images/cmu15445-project4/3.png)
![img](../images/cmu15445-project4/4.png)

> MySQL InnoDB引擎的默认隔离级别虽然是[可重复读]，但是它很大程度上避免幻读现象，并没有解决幻读 [参考文章](https://xiaolincoding.com/mysql/transaction/phantom.html)<br>
***
> 解决方案有两种:
> - 普通select语句(快照读),通过MVCC方式解决了幻读问题
> - select ... for update语句(当前读)，通过next-key lock(记录锁 + 间隙锁)方式解决了幻读

### Read View在MVCC里如何工作的?

Read View有四个重要的字段<br>
四个字段只有m_ids是一个集合，creator_trx_id在m_ids集合中，
min_trx_id是m_ids集合的最小值
![img](../images/cmu15445-project4/5.png)

记录的两个隐藏列
![img](../images/cmu15445-project4/6.png)

- trx_id，当一个事务对某条聚簇索引记录进行改动时，就会把该事务的事务 id 记录在 trx_id 隐藏列里；
- roll_pointer，每次对某条聚簇索引记录进行改动时，都会把旧版本的记录写入到 undo 日志中，然后这个隐藏列是个指针，指向每一个旧版本记录，于是就可以通过它找到修改前的记录

在创建ReadView后，一个事务去访问记录的时候，除了自己的更新记录总是可见之外，还有几种情况：
- 如果记录的trx_id小于ReadView的min_trx_id，表示这个版本的记录是在创建 Read View 前已经提交的事务生成的，所以该版本的记录对当前事务可见
- 如果记录的trx_id大于等于ReadView中的max_trx_id,表示这个版本的记录是在创建 Read View 后才启动的事务生成的，所以该版本的记录对当前事务不可见
- 如果记录的 trx_id 值在 Read View 的 min_trx_id 和 max_trx_id 之间，需要判断 trx_id 是否在 m_ids 列表中：
    - 如果记录的 trx_id 在 m_ids 列表中，表示生成该版本记录的活跃事务依然活跃着（还没提交事务），所以该版本的记录对当前事务不可见。
    - 如果记录的 trx_id 不在 m_ids列表中，表示生成该版本记录的活跃事务已经被提交，所以该版本的记录对当前事务可见。


### Two Phase Locking 两阶段锁
2PL将事务划分为两个阶段:
- Growing Phase: 只获得锁
- Shrink Phase: 只释放锁

![img](../images/cmu15445-project4/7.png)
2PL本身已经足够保证schedule是seriable的，但2PL可能导致cascading aborts，举例如下:
![img](../images/cmu15445-project4/8.png)

于是引入2PL的增强版变种，Rigorous 2PL，后者每个事务在结束之前，其写过的数据不能被其它事务读取或者重写

![img](../images/cmu15445-project4/9.png)

### Deadlock Detection & Prevention
2PL 无法避免的一个问题就是死锁，解决方案：
#### Deadlock Detection 事后检测
为了检测死锁，DBMS 会维护一张 waits-for graph，来跟踪每个事务正在等待 (释放锁) 的其它事务，然后系统会定期地检查 waits-for graph，看其中是否有成环，如果成环了就要决定如何打破这个环。<br>
waits-for graph 中的节点是事务，从 Ti 到 Tj 的边就表示 Ti 正在等待 Tj 释放锁，举例如下
![img](../images/cmu15445-project4/10.png)
当 DBMS 检测到死锁时，它会选择一个 "受害者" (事务)，将该事务回滚，打破环形依赖，而这个 "受害者" 将依靠配置或者应用层逻辑重试或中止。这里有两个设计决定：
1. 检测死锁的频率
2. 如何选择合适的 "受害者"

检测死锁的频率越高，陷入死锁的事务等待的时间越短，但消耗的 cpu 也就越多。所以这是个典型的 trade-off，通常有一个调优的参数供用户配置。

选择 "受害者" 的指标可能有很多：事务持续时间、事务的进度、事务锁住的数据数量、级联事务的数量、事务曾经重启的次数等等。在选择完 "受害者" 后，DBMS 还有一个设计决定需要做：完全回滚还是回滚到足够消除环形依赖即可。

#### Deadlock Prevention 事前阻止
通常 prevention 会按照事务的年龄来赋予优先级，事务的时间戳越老，优先级越高。有两种 prevention 的策略： <br>
- Old Waits for Young：如果 requesting txn 优先级比 holding txn 更高则等待后者释放锁；更低则自行中止<br>
- Young Waits for Old：如果 requesting txn 优先级比 holding txn 更高则后者自行中止释放锁，让前者获取锁，否则 requesting txn 等待 holding txn 释放锁

举例如下:

![img](../images/cmu15445-project4/11.png)
