这部分放一些查询优化和 MySQL 查询优化的内容，偏向一些实现细节

## Multi-Range Optimization

hedengcheng 的 slide：https://github.com/hedengcheng/tech/blob/master/database/MySQL/MySQL%E6%9F%A5%E8%AF%A2%E4%BC%98%E5%8C%96%E6%B5%85%E6%9E%90.pdf

(强烈推荐) MySQL源码：Range优化相关的数据结构: https://www.orczhou.com/index.php/2012/11/mysql-source-code-range-optimize-data-structure/#i 和 https://www.orczhou.com/index.php/2013/01/mysql-source-code-range-optimize-data-structure-again/

Range (Min-Max Tree ) 结构分析：http://mysql.taobao.org/monthly/2021/06/03/