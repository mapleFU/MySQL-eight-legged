下面内容的来源：

* 官方的博客: MariaDB / MySQL \(https://dev.mysql.com/blog-archive/\)
* 数据库月报: https://github.com/tangwz/db-monthly
* PolarDB-X 团队知乎: https://www.zhihu.com/org/polardb-x
* skywalker 的文章: https://www.zhihu.com/people/jiang-feng-73-84/posts
* http://liuyangming.tech/

代码目录：https://blog.csdn.net/qq_16668303/article/details/111765310 （要不然鬼看得懂）

## Innodb_buffer_pool

链接：

1. **(强烈推荐)** InnoDB Buffer Pool: http://mysql.taobao.org/monthly/2017/05/01/

2. InnoDB BufferPool 浅析: http://mysql.taobao.org/monthly/2020/02/02/

3. InnoDB Buffer Pool flush 特性漫谈：http://mysql.taobao.org/monthly/2015/02/01/  && InnoDB 的自适应刷脏：https://leviathan.vip/2020/05/19/mysql-understand-adaptive-flushing/

4. MySQL · 内核分析 · InnoDB Buffer Pool 并发控制 http://mysql.taobao.org/monthly/2020/05/06/
	1. (内存足够的情况下) 会有多个 Buffer Pool Instance。用来拆小并发。8.0.13 之后 apply 了 Percona 提交上来的更改，将锁的并发粒度拆细了
	2. page 上会有 `io_fix` (io 状态) 和 Pin 的状态，根据这些启发式的避免一些 Lock
	3. Flush 的时候会上 SX Lock

5. **(强烈推荐)** MySQL · 引擎特性 · PolarDB Innodb刷脏优化 http://mysql.taobao.org/monthly/2023/04/02/ 

   1. 这部分内容可以联动 「MySQL InnoDB读写锁实现分析 - 阿里云数据库开源的文章 - 知乎 https://zhuanlan.zhihu.com/p/408637966 」 和 「InnoDB 表空间压缩」 + IO 子系统一起看
   2. InnoDB 有 LRU 和 Flush List. LRU 是一个 3/8 切分的类似 2-LRU 的结构，Flush List 按照 modification 进行排序（见 InnoDB 8.0 的更新）
   3. 会有拆的比较细的锁。
   4. SSD 的 Page Flush 的 IO 可能比较快，所以这块相对来说开销不是很离谱。PolarDB 在的环境会做一轮 Shadow Flush（类似 CoW）来阻止对应的开销
   5. InnoDB 在 Flush List 中的页面也会在 LRU List 中，在需要内存之类的时候，会有不同的策略来 Flush，同时，InnoDB 自己会有 Page Cleaner 线程去 Trigger 自适应的 Flushing，根据 Redo 产生数据、Dirty Page Ratio 之类的来 Flush，ckpt 之类提交的时候可能按照配置会需要 WAL Sync 到某个位点。这个时候都会 Trigger Flush。内存压力大的时候，可能会用户线程主动触发 Flush

一些细节:

1. Chunk 和内存申请
2. Zip 的使用 和 Buddy System
3. 和 Redo Log 空间/IO 能力/Dirty Page 的联动
4. LRU List 的策略和调整，包括 `LRU_Manager_Thread` 和 `Page_Cleaner_Thread`
5. DoubleWrite
6. 预加载

顺便说一句，Double Write 可以联动 PG 的 full page write 看：http://mysql.taobao.org/monthly/2015/11/05/

NVMe 最新协议似乎有原子写有关的部分。主线程有一些行为可以联动一下。

关注：

1. 限制
2. key 和 value 的 pattern.

## 存储(关注 Tablespace -> Page 的链路，不关注逻辑)

1. (强烈推荐) 官方博客：https://dev.mysql.com/blog-archive/innodb-tablespace-space-management/ ，有几张绝世好图
2. (强烈推荐) 《MySQL是怎样运行的：从根儿上理解 MySQL》 chap8-9
3. http://mysql.taobao.org/monthly/2019/10/01/
4. (强烈推荐) https://dev.mysql.com/doc/refman/8.0/en/innodb-compression-internals.html InnoDB Table Compression 的内部逻辑，推荐和 Page Compression 部分联动看

基本上是为了让分配的物理空间连续/不连续导致的

Page 存储感觉做的非常细，相对别的存储引擎，对 update 之类的更新操作做了很深入的考虑，能感受到引擎的高效和权衡。

### IO 特性: Extend 层次上的预读

Extend 可能会（使 workload 和配置而定）：

* 如果线性扫描 Extends，可能会预读后面的 Extend
* Extend 内 Page 访问满足一定条件，会加载起这个 Extend 内的数据

具体可以参考：

* 线性预读：https://bbs.huaweicloud.com/blogs/107748
*  MySQL · 答疑解惑 · InnoDB 预读 VS Oracle 多块读 http://mysql.taobao.org/monthly/2015/05/04/

## 分区

* http://mysql.taobao.org/monthly/2017/11/09/

个人还是关注 `Null` 特性，和 OLAP 使用上的改进。

## Adaptive Hash Index

看来是 Page 有关的，我觉得设计的很巧妙。但是不知道占用空间之类的怎么说

1. http://mysql.taobao.org/monthly/2015/09/01/
2. https://dev.mysql.com/doc/refman/8.0/en/innodb-adaptive-hash.html

感觉是 mem + page 的一个系统，我觉得非常巧妙。

## btr

关于 cluster index 之类的我觉得没啥好讲的，HeapTable 之类的可以对比，但是不能乱碰瓷。

下面两篇虽然是实现，但是是**强烈推荐**的内容，因为写的太牛逼了：

* InnoDB：B-tree index（1） - Skywalker的文章 - 知乎 https://zhuanlan.zhihu.com/p/164705538
* InnoDB：B-tree index（2） - Skywalker的文章 - 知乎 https://zhuanlan.zhihu.com/p/164728032

### Page

1. 索引Page 内的逻辑，包括一些插入 + 方向优化：http://mysql.taobao.org/monthly/2018/04/03/
2. （比较简单）Merge 和 Split 的一些要点：https://www.percona.com/blog/2017/04/10/innodb-page-merging-and-page-splitting/
3. https://stackoverflow.com/questions/48364549/how-does-the-leaf-node-split-in-the-physical-space-in-innodb
4. 压缩页：http://mysql.taobao.org/monthly/2015/08/01/
   1. 压缩的内容有一篇官方博客：https://dev.mysql.com/blog-archive/innodb-transparent-page-compression/
   2. Percona 有一篇博客 MySQL: Compression, Indexes, and Two Smoking Barrels https://www.percona.com/blog/mysql-compression-indexes-and-two-smoking-barrels/
   3. 数据库月报还提供了一个压缩相关的博客，提到了 Percona 的列存插件  MySQL · 引擎特性 · Column Compression浅析: http://mysql.taobao.org/monthly/2016/11/04/

感觉很细的一点是，根据插入方向确定 split 的 pattern，和页内格式。我反正写不了这么复杂的东西。

Split 大概逻辑在 `btr_page_split_and_insert`, 下面两篇文章链路介绍的比较好：

1. InnoDB——Btree与MTR的牵扯: http://liuyangming.tech/05-2019/InnoDB-Mtr.html

2. MySQL 8.0 redo log实现分析: https://zhuanlan.zhihu.com/p/440476383

3. InnoDB B-tree 顺序插入优化及问题 https://zhuanlan.zhihu.com/p/398795379

下降的实现在 `btr_cur_search_to_nth_level`，插入之类的都会走这里。

1. MySQL · 源码分析 · btr_cur_search_to_nth_level 函数分析: http://mysql.taobao.org/monthly/2021/07/02/

#### 并行读

MySQL 8.0 引入了并行读的框架。具体：http://mysql.taobao.org/monthly/2019/10/03/ && https://www.percona.com/blog/2019/01/23/mysql-8-0-14-a-road-to-parallel-query-execution-is-wide-open/

* InnoDB 并行读取框架 https://leviathan.vip/2020/10/02/innodb-parallel-read-of-index/

## 统计信息

1. (强烈推荐) MySQL · 内核特性 · 统计信息的现状和发展 http://mysql.taobao.org/monthly/2020/12/05/

2. MySQL · 内核分析 · InnoDB 的统计信息 http://mysql.taobao.org/monthly/2020/03/08/

3. MySQL 深潜 - 统计信息采集 http://mysql.taobao.org/monthly/2022/10/05/ （这篇文章比较详细描述了 btr 中采样 ndv 等 item 的实现，比之前细不少）

## Record

1. (强烈推荐) 《MySQL是怎样运行的：从根儿上理解 MySQL》

（俺实现过一遍 MySQL Compact，性能还真挺好的）

### BLOB

* MySQL · 源码分析 · innodb-BLOB演进与实现 http://mysql.taobao.org/monthly/2022/09/01/

### JSON

1. MySQL5.7 的 JSON 实现 http://mysql.taobao.org/monthly/2016/01/03/
2. 如何索引 JSON 字段 http://mysql.taobao.org/monthly/2017/12/09/

### Decimal

1. http://mysql.taobao.org/monthly/2021/03/02/

### Multi-Values Index

InnoDB 的索引，对同一个 Primary Key，建立多个二级索引。相当于从虚拟列（ JSON ) 中抽取对应的实现。

* MySQL · 引擎特性 · Multi-Valued Indexes 简述 http://mysql.taobao.org/monthly/2019/09/04/ 

### 编码

Varchar 之类的处理。

## Latch

### Latch 的物理实现(其实很 naive)

这个 SpinLock 实现的时候竟然会 random 一下，详细如下

* https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-spin_lock_polling.html

然后它 Latch ：http://mysql.taobao.org/monthly/2020/03/07/  和 http://mysql.taobao.org/monthly/2021/02/08/

### Latch Protocol (挺难的我觉得...)

本身 Btr 有一些 Latching Protocol, 这个是个很大的话题

1. (强烈推荐) http://mysql.taobao.org/monthly/2022/01/01/
2. (强烈推荐) https://zhuanlan.zhihu.com/p/151397269
3. MySQL · 引擎特性 · InnoDB index lock前世今生 http://mysql.taobao.org/monthly/2015/07/05/

这个时候我不得不推荐一下 B-link-Tree 的解析了：https://zhuanlan.zhihu.com/p/165149237

## 锁

MySQL 锁有很多坑，介绍最好的材料应该是何登成写的：

1. https://github.com/hedengcheng/tech/blob/master/database/MySQL/MySQL%20%E5%8A%A0%E9%94%81%E5%A4%84%E7%90%86%E5%88%86%E6%9E%90.pdf
2. https://github.com/wiminq/tech_note/blob/master/MySQL/%E4%BD%95%E7%99%BB%E6%88%90PPT/InnoDB%20Transaction%20Lock%20and%20MVCC%20%252854%E9%A1%B5%2529.pdf

官方博客在 20 年给了几篇博客：

* InnoDB Data Locking - Part 1 "Introduction" https://dev.mysql.com/blog-archive/innodb-data-locking-part-1-introduction/
* InnoDB Data Locking - Part 2 "Locks" https://dev.mysql.com/blog-archive/innodb-data-locking-part-2-locks/
* InnoDB Data Locking - Part 2.5 "Locks" (Deeper dive) https://dev.mysql.com/blog-archive/innodb-data-locking-part-2-5-locks-deeper-dive/
* InnoDB Data Locking – Part 3 "Deadlocks" https://dev.mysql.com/blog-archive/innodb-data-locking-part-3-deadlocks/
* InnoDB Data Locking - Part 4 "Scheduling" https://dev.mysql.com/blog-archive/innodb-data-locking-part-4-scheduling/
* InnoDB Data Locking - Part 5 "Concurrent queues" https://dev.mysql.com/blog-archive/innodb-data-locking-part-5-concurrent-queues/

MySQL 博客给的文章也很不错：

* MySQL · 引擎特性 · InnoDB 事务锁系统简介 http://mysql.taobao.org/monthly/2016/01/01/
* MySQL · 性能优化 · InnoDB 事务 sharded 锁系统优化 http://mysql.taobao.org/monthly/2021/02/04/

注意

1. 一些 insertion 的意向锁 之类的。
2. 不支持锁 Escalation (不同于 upgrade)，虽然有 intention lock，但是那个可以理解成表锁之类的。但 InnoDB 支持锁表更新 `(lock_rec_inherit_to_gap_if_gap_lock)`
3. 实现类似 wound-wait 的 2pl，lock 和 page 挂钩
4. InnoDB 锁的内存结构（虽然它就是仅内存的）：全局管理，记录锁和 index/page 有关联，会针对 page 有一个 bitmap。

这里读还区分了 consistent nonlocking read 和 locking read，其实还有个奇怪的 semi-consistent read

还有个 auto_increment: https://dev.mysql.com/doc/refman/8.0/en/innodb-auto-increment-handling.html

关于锁，还有一些死锁检测的内容：

* MySQL · 引擎特性 · 死锁检测 http://mysql.taobao.org/monthly/2021/05/02/

隔离级别和锁还是看何登成的材料吧。

### CATS / VATS

InnoDB 里面的锁调度策略，有一些相关论文支持，大概意思是先调度有更多 trxn 等待的 trxn。

* (强烈推荐) MySQL 8 事务锁调度VATS简介 https://zhuanlan.zhihu.com/p/412672250
* MySQL · 源码分析 · 事务锁调度分析 http://mysql.taobao.org/monthly/2021/09/01/

## 事务子系统

* (强烈推荐) https://zhuanlan.zhihu.com/p/365415843 InnoDB事务 - 从原理到实现（zty 老板写的）

* (强烈推荐) MairaDB Slide: https://mariadb.org/wp-content/uploads/2018/02/Deep-Dive_-InnoDB-Transactions-and-Write-Paths.pdf

* MySQL · 引擎特性 · InnoDB 事务子系统介绍: http://mysql.taobao.org/monthly/2015/12/01/

* MySQL · 引擎特性 · InnoDB mini transation: http://mysql.taobao.org/monthly/2017/10/03/

  * MySQL · 源码详解 · mini transaction详解 http://mysql.taobao.org/monthly/2021/09/04/ (这篇似乎是上一篇的续篇，介绍的更详细一些)

Redo/Undo 强烈推荐 Catkang 的 notes:

1. https://catkang.github.io/2020/02/27/mysql-redo.html
2. https://catkang.github.io/2021/10/30/mysql-undo.html

Undo 还有下面这篇：

1. (强烈推荐) MySQL · 源码解析 · InnoDB中undo日志的组织及实现  http://mysql.taobao.org/monthly/2023/05/01/

关注：

1. undo log 的实现 pattern
2. Physical/Logical 的物理逻辑日志，和 column level 的操作
3. **Redo Log 的 lock-free 写模型**
4. MVCC 怎么和事务/索引/Undo Log 联动的
5. UNDO LOG 的 GC，和事务信息系统

一些细节的代码可以参考：

1. MySQL · 源码分析 · LinkBuf设计与实现 http://mysql.taobao.org/monthly/2019/05/08/
	2. MySQL Link buf - 阿里云数据库开源的文章 - 知乎https://zhuanlan.zhihu.com/p/408569476
2. 源码分析 · InnoDB Redo Log 重构 http://mysql.taobao.org/monthly/2022/09/03/
   1. 这篇的官方文档在：https://blogs.oracle.com/mysql/post/dynamic-innodb-redo-log-in-mysql-80
3. InnoDB Redo 日志归档(8.0.17 提出)：
   1. 阿里云的使用 https://developer.aliyun.com/article/807030
   2. 官方博客：https://dev.mysql.com/blog-archive/mysql-innodb-redo-log-archiving/
4. (强烈推荐) (推荐看完 CatKang 那篇回来看) InnoDB：redo log（1） - Skywalker的文章 - 知乎 https://zhuanlan.zhihu.com/p/386710765 
5. (强烈推荐 + 官方博客) https://dev.mysql.com/blog-archive/mysql-8-0-new-lock-free-scalable-wal-design/ 
   * 这一篇是它的翻译 http://mysql.taobao.org/monthly/2018/06/01/
6. (5.7 版本对比实现，可以对比前面几篇读) InnoDB——LogBuffer与事务提交过程 http://liuyangming.tech/06-2019/LogBufferAndBufferPool.html
7. InnoDB MVCC 相关实现 - 阿里云数据库开源的文章 - 知乎
    https://zhuanlan.zhihu.com/p/414088892 (关注 read view, 事务和 Cluster Index / 非 Cluster Index 的锁)
  1. InnoDB——Btree与rwlock的互动 http://liuyangming.tech/07-2019/InnoDB-Lock.html
  2. MySQL · 引擎特性 · InnoDB MVCC 相关实现 http://mysql.taobao.org/monthly/2018/11/04/

#### Undo 

Undo 可以关注 undo tablespace 的创建和管理

1. MySQL 5.6 引入，回收 undo 的空间：https://dev.mysql.com/blog-archive/online-truncate-of-innodb-undo-tablespaces/
2. MySQL 8.0 之后，对上述功能增强：
   1. https://dev.mysql.com/blog-archive/mysql-8-0-2-more-flexible-undo-tablespace-management/
   2. https://dev.mysql.com/blog-archive/new-in-mysql-8-0-14-create-undo-tablespace/

阿里数据库团队有人写过相关的材料，看了下，这哥们写了非常多 Undo 的，比较靠谱，可以翻他的 posts:

* InnoDB之UNDO LOG介绍 - 就是酱的文章 - 知乎 https://zhuanlan.zhihu.com/p/453169285

* Undo TableSpace 的发展: http://mysql.taobao.org/monthly/2020/10/02/

还有一些别的评论：

* Long-lived Transactions 产生的影响 http://mysql.taobao.org/monthly/2023/02/03/  (以工程师的视角介绍了一下，内容可以参考 SAP HANA 和 HyPer 的 Interval GC)

### binlog

关于 Binlog, 这玩意还挺重要的，这里有：

1. Binlog 本身内容
2. Binlog 和 Redo/Undo/XA 的联动
3. Group Commit

推荐的材料是：

1. \<redo、undo、buffer pool、binlog，谁先谁后， 有点儿乱\>

2. 无处不在的 MySQL XA 事务: https://zhuanlan.zhihu.com/p/372300181

3. MySQL Binlog 源码入门 http://mysql.taobao.org/monthly/2023/01/04/

### XA

XA 概念和锁强相关，本身可以先看看上面 Binlog 有关的。这里推荐 PolarDB-X 关于 XA 的文章，介绍了 XA 及其优化。

* https://zhuanlan.zhihu.com/p/355413022

### Recover

Recover 本身涉及 undo/redo，事务的状态也要由 XA 来决定。

在 InnoDB 的层面, commit 相当于在 undo data 上标记 Undo Segment Header 中的State会被从TRX_UNDO_ACTIVE改成TRX_UNDO_TO_FREE，TRX_UNDO_TO_PURGE或TRX_UNDO_CACHED, 然后写 mtr 的 redo log。同时，如果开启了 XA，可能要根据 XA / binlog 来恢复。这里还有一些 checkpoint 相关的内容，来避免 Redo 过长，加速恢复。相关博客如下：

1. MySQL · 引擎特性 · InnoDB崩溃恢复: http://mysql.taobao.org/monthly/2017/07/01/
2. MySQL · 引擎特性 · InnoDB 崩溃恢复过程: http://mysql.taobao.org/monthly/2015/06/01/

上面两个是 undo/redo 层面的，如果事务状态是 PREPARE，而且开启了 XA，需要考虑：

1. MySQL · 源码分析 · binlog crash recovery: http://mysql.taobao.org/monthly/2018/07/05/
2. mysql源码学习笔记：基于binlog的recovery机制 https://blog.csdn.net/slwang001/article/details/77343893
3. (一个很有趣的例子，介绍 XA Recover 的锁处理) [MySQL 分布式事务锁恢复机制探究](https://zhuanlan.zhihu.com/p/50762037)

下面有一篇比较新的优化趋势优化的文章，即 binlog in redo:

1. AliSQL · 内核特性 · Binlog In Redo: http://mysql.taobao.org/monthly/2020/06/01/

一些细节：

* 【日常技术批判】 InnoDB 确定 checkpoint-lsn 的一处细节 https://github.com/rsy56640/triviality/tree/master/content/innodb-ckpt-lsn


## Change Buffer/Insert Buffer

链接：

1. http://mysql.taobao.org/monthly/2015/07/01/
2. https://dev.mysql.com/doc/refman/8.0/en/innodb-change-buffer.html
3. 官方博客：https://dev.mysql.com/blog-archive/the-innodb-change-buffer/
4. InnoDB: Change Buffer https://zhuanlan.zhihu.com/p/346500273
