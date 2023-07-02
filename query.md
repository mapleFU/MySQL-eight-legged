这部分放一些查询优化和 MySQL 查询优化的内容，偏向一些实现细节

## Multi-Range Optimization

hedengcheng 的 slide：https://github.com/hedengcheng/tech/blob/master/database/MySQL/MySQL%E6%9F%A5%E8%AF%A2%E4%BC%98%E5%8C%96%E6%B5%85%E6%9E%90.pdf

(强烈推荐) MySQL源码：Range优化相关的数据结构: https://www.orczhou.com/index.php/2012/11/mysql-source-code-range-optimize-data-structure/#i 和 https://www.orczhou.com/index.php/2013/01/mysql-source-code-range-optimize-data-structure-again/

Range (Min-Max Tree ) 结构分析：http://mysql.taobao.org/monthly/2021/06/03/

## 一些入口

* 引擎层入口(这里还涉及一些 XA 的逻辑)

  * MySQL · 内核特性 · Attachable transaction http://mysql.taobao.org/monthly/2020/06/03/
  * MySQL · 源码阅读 · 内部XA事务 http://mysql.taobao.org/monthly/2021/01/02/
  * https://www.codeproject.com/Articles/1107279/Writing-a-MySQL-Storage-Engine-from-Scratch

## 特定查询的优化

* Count(*) http://mysql.taobao.org/monthly/2023/02/01/ 在 5.6 和 8.x 版本之后，会下推到存储引擎，新版本的代码会扫 Cluster Index，然后尽量扫的数据不进 buffer-pool
* 子查询的执行: MySQL · 源码解析 · mysql 子查询执行方式介绍 http://mysql.taobao.org/monthly/2023/06/02/