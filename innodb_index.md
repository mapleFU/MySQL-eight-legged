

代码目录：https://blog.csdn.net/qq_16668303/article/details/111765310 （要不然鬼看得懂）

## Innodb_buffer_pool

链接：

1. **(强烈推荐)** InnoDB Buffer Pool: http://mysql.taobao.org/monthly/2017/05/01/
2. BufferPool 浅析: http://mysql.taobao.org/monthly/2020/02/02/ 
3. InnoDB Buffer Pool 特性漫谈：http://mysql.taobao.org/monthly/2015/02/01/

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

基本上是为了让分配的物理空间连续/不连续导致的

Page 存储感觉做的非常细，相对别的存储引擎，对 update 之类的更新操作做了很深入的考虑，能感受到引擎的高效和权衡。

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

感觉很细的一点是，根据插入方向确定 split 的 pattern，和页内格式。我反正写不了这么复杂的东西。

Split 大概逻辑在 `btr_page_split_and_insert`, 下面两篇文章链路介绍的比较好：

1. InnoDB——Btree与MTR的牵扯: http://liuyangming.tech/05-2019/InnoDB-Mtr.html
2. MySQL 8.0 redo log实现分析: https://zhuanlan.zhihu.com/p/440476383

下降的实现在 `btr_cur_search_to_nth_level`，插入之类的都会走这里。

1. MySQL · 源码分析 · btr_cur_search_to_nth_level 函数分析: http://mysql.taobao.org/monthly/2021/07/02/

### DDL

Online DDL 对业务来说还是很重要的。

1. (一些演进) Online DDL 演进： http://mysql.taobao.org/monthly/2021/03/06/
2. (这个功能好像是腾讯/阿里开发的，看到 Percona 还是啥写过感谢) 快速加列：http://mysql.taobao.org/monthly/2020/03/01/

感觉没找到很好的材料，需要区分 online or not, in-place or not, rebuild or not. 然后 gh-ost/pt-online-schema-change 之类的又有一些 DDL:

1. http://mysql.taobao.org/monthly/2018/05/02/

反正这块还挺复杂的，突然发现 TiDB 那套模型加索引还挺方便的 orz。

## 统计信息

1. http://mysql.taobao.org/monthly/2020/12/05/
2. http://mysql.taobao.org/monthly/2020/03/08/


## Record

1. (强烈推荐) 《MySQL是怎样运行的：从根儿上理解 MySQL》

（俺实现过一遍 MySQL Compact，性能还真挺好的）

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

注意

1. 一些 insertion 的意向锁 之类的。
2. 不支持锁 Escalation (不同于 upgrade)，虽然有 intention lock，但是那个可以理解成表锁之类的
3. 实现类似 wound-wait 的 2pl，lock 和 page 挂钩

这里读还区分了 consistent nonlocking read 和 locking read，其实还有个奇怪的 semi-consistent read

还有个 auto_increment: https://dev.mysql.com/doc/refman/8.0/en/innodb-auto-increment-handling.html

关于锁，还有一些死锁检测的内容：http://mysql.taobao.org/monthly/2021/05/02/

隔离级别和锁还是看何登成的材料吧。

## 事务子系统

* (强烈推荐) https://zhuanlan.zhihu.com/p/365415843 InnoDB事务 - 从原理到实现（zty 老板写的）

* MySQL · 引擎特性 · InnoDB 事务子系统介绍: http://mysql.taobao.org/monthly/2015/12/01/

* MySQL · 引擎特性 · InnoDB mini transation: http://mysql.taobao.org/monthly/2017/10/03/

Redo/Undo 强烈推荐 Catkang 的 notes:

1. https://catkang.github.io/2020/02/27/mysql-redo.html
2. https://catkang.github.io/2021/10/30/mysql-undo.html

关注：

1. undo log 的实现 pattern
2. Physical/Logical 的物理逻辑日志，和 column level 的操作
3. **Redo Log 的 wait-free 写模型**
4. MVCC 怎么和事务/索引/Undo Log 联动的
5. UNDO LOG 的 GC，和事务信息系统

关于 Binlog, 这玩意还挺重要的，这里有：

1. Binlog 本身内容
2. Binlog 和 Redo/Undo/XA 的联动
3. Group Commit

推荐的材料是：

1. \<redo、undo、buffer pool、binlog，谁先谁后， 有点儿乱\>
2. https://zhuanlan.zhihu.com/p/372300181

### XA 

XA 概念和锁强相关，本身可以先看看上面 Binlog 有关的。这里推荐 PolarDB-X 关于 XA 的文章，介绍了 XA 及其优化。

* https://zhuanlan.zhihu.com/p/355413022

## 备份

1. http://mysql.taobao.org/monthly/2016/03/07/
2. http://mysql.taobao.org/monthly/2015/08/09/ 和 http://mysql.taobao.org/monthly/2015/09/07/

## Change Buffer/Insert Buffer

链接：
1. http://mysql.taobao.org/monthly/2015/07/01/
2. https://dev.mysql.com/doc/refman/8.0/en/innodb-change-buffer.html
3. 官方博客：https://dev.mysql.com/blog-archive/the-innodb-change-buffer/
