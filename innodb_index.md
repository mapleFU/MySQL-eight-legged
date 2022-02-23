

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

1. (强烈推荐) 《MySQL是怎样运行的：从根儿上理解 MySQL》 chap8-9
2. http://mysql.taobao.org/monthly/2019/10/01/

基本上是为了让分配的物理空间连续/不连续导致的

Page 存储感觉做的非常细，相对别的存储引擎，对 update 之类的更新操作做了很深入的考虑，能感受到引擎的高效和权衡。

## 分区

* http://mysql.taobao.org/monthly/2017/11/09/

个人还是关注 `Null` 特性，和 OLAP 使用上的改进。

## btr


## Record

1. (强烈推荐) 《MySQL是怎样运行的：从根儿上理解 MySQL》

（俺实现过一遍 MySQL Compact，性能还真挺好的）

### 编码

Varchar 之类的处理。

## Change Buffer/Insert Buffer

链接：
1. http://mysql.taobao.org/monthly/2015/07/01/
2. https://dev.mysql.com/doc/refman/8.0/en/innodb-change-buffer.html





secondary / unique