## Replication && DDL && Cloud

### Replication

1. (强烈推荐) MySQL · 源码分析 · MySQL 半同步复制数据一致性分析 http://mysql.taobao.org/monthly/2017/04/01/

#### 备份

1. MySQL · 物理备份 · Percona XtraBackup 备份原理 http://mysql.taobao.org/monthly/2016/03/07/

2. MySQL · 功能分析 · 5.6 并行复制实现分析 http://mysql.taobao.org/monthly/2015/08/09/ 和 MySQL · 特性分析 · 5.6 并行复制恢复实现 http://mysql.taobao.org/monthly/2015/09/07/


### DDL

Online DDL 对业务来说还是很重要的。

1. (一些演进) Online DDL 演进： http://mysql.taobao.org/monthly/2021/03/06/
2. (这个功能好像是腾讯/阿里开发的，看到 Percona 还是啥写过感谢) 快速加列：http://mysql.taobao.org/monthly/2020/03/01/

感觉没找到很好的材料，需要区分 online or not, in-place or not, rebuild or not. 然后 gh-ost/pt-online-schema-change 之类的又有一些 DDL:

1. http://mysql.taobao.org/monthly/2018/05/02/

反正这块还挺复杂的，突然发现 TiDB 那套模型加索引还挺方便的 orz。

#### Atomic DDL

TiDB/F1 的 online ddl 参考了论文 `<Online, Asynchronous Schema Change in F1>`，在分布式机器中做了一个状态机后推的兼容性。然后 DDL 靠事务进行。`gh-ost` 会考虑相关的 DDL 的兼容和追 lag 什么的，这里 MySQL 8.0 和 InnoDB 协同，来做了一些 Atomic DDL 相关的内容：

1. 官方博客:

   1. Data Directory 的 motivation： https://dev.mysql.com/blog-archive/mysql-8-0-data-dictionary-background-and-motivation/
   2. Data Directory 的设计：https://dev.mysql.com/blog-archive/mysql-8-0-data-dictionary-architecture-and-design/
   3. Atomic DDL: https://dev.mysql.com/blog-archive/atomic-ddl-in-mysql-8-0/

2. 行为：

   1. 深入解读MySQL8.0 新特性 ：Crash Safe DDL： https://developer.aliyun.com/article/692258
   2. MySQL · 源码分析 · DDL log与原子DDL的实现 http://mysql.taobao.org/monthly/2021/07/05/
3. MySQL · 源码分析 · 8.0 · DDL的那些事 http://mysql.taobao.org/monthly/2020/05/05/
4. MySQL · 源码分析 · 原子DDL的实现过程 http://mysql.taobao.org/monthly/2018/03/02/
5. MySQL · 源码分析 · 8.0 原子DDL的实现过程续 http://mysql.taobao.org/monthly/2018/07/02/


## Cloud DB

* (强烈推荐) 网易数帆MySQL云原生数据库实现 - 温正湖的文章 - 知乎
https://zhuanlan.zhihu.com/p/547171082 -- 这篇既不是第一个做的，网易也不是 cloud db 做的最好的(甚至差很远)，但这篇很详细的描述了 MySQL 上云的问题和解决方案，相当推荐