# MySQL-eight-legged
eight-legged essay for MySQL. Just a learning Project

## Materials

1. https://book.douban.com/subject/25872763/
2. https://book.douban.com/subject/24708143/
3. https://book.douban.com/subject/35231266/
4. 官方文档：https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html
5. MariaDB 文档：https://mariadb.com/kb/en/mysqld_safe/

本来官方文档应该是最好的选择，但是官方文档太垃圾了，以至于要看很多二手文档。

Alibaba 数据库月报很牛逼，但是看八股文的时候不太想碰源代码，以后再说。

## Setting up

应该先设置 docker:

```
docker run -itd --name mysql-test-ev -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```

这里用了 mysql 的镜像，拉取的是最新版本：

https://hub.docker.com/_/mysql

有需求的话可以拉想要的版本.

连接：

```
mysql --host 127.0.0.1 --port 4001 -u root -p
```

进 shell:

```
docker exec -it mysql-test-ev /bin/bash 
```

## Common files and config

在 shell 里，可以找到 MySQL 的文件和配置：

```
root@c9fb2a2b2402:/etc# cd mysql
root@c9fb2a2b2402:/etc/mysql# ls
conf.d	my.cnf	my.cnf.fallback
```

还有个：

```
./etc/mysql/conf.d/mysql.cnf
```

MySQL 和 MySQL 客户端启动的时候，能去这里

然后 bin:

```
root@c9fb2a2b2402:/usr/bin# ls | grep mysql
mysql
mysql_config_editor
mysql_secure_installation
mysql_ssl_rsa_setup
mysql_tzinfo_to_sql
mysql_upgrade
mysqladmin
mysqlbinlog
mysqlcheck
mysqld_multi
mysqld_safe
mysqldump
mysqldumpslow
mysqlimport
mysqlpump
mysqlshow
mysqlslap
```

https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html 这里附近大概介绍了下这些 binary 都是做什么的。

比如 mysql_multi 能够起多个，mysqld_safe 能在 mysqld 挂了给你保存日志然后捞一份。

`show engines` 可以看到 engines, 这里还有一些选项：

1. Engine
2. Support
3. Comment
4. Transactions
5. XA
6. Savepoints



然后我们说回 config, MySQL 有启动变量，系统变量(`VARIABLES`)，状态变量 `STATUS`

启动变量就是很正常的启动配置，需要注意的是这甚至可以调内存分配器

系统变量很多可以运行时更改，区分了 GLOBAL 和 SESSION，默认是 session.

Status 如下：

```
mysql> show status like 'thread%'
    -> ;
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 0     |
| Threads_connected | 1     |
| Threads_created   | 1     |
| Threads_running   | 2     |
+-------------------+-------+
4 rows in set (0.01 sec)
```





