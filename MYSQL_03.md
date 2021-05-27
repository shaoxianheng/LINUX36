



# MYSQL 03

## 1.1.1 查看students表的索引

```
MariaDB [hellodb]> show indexes from students\G
*************************** 1. row ***************************
        Table: students
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: StuID
    Collation: A
  Cardinality: 25
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
1 row in set (0.00 sec)

```

## 1.1.2 explain 查看是否使用索引

```
MariaDB [hellodb]> explain select * from students where age=20;
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
| id   | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
|    1 | SIMPLE      | students | ALL  | NULL          | NULL | NULL    | NULL |   25 | Using where |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)

```

## 1.1.3 创建索引

```
MariaDB [hellodb]> create index idx_age on students(age);
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [hellodb]> show indexes from students;
+----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table    | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| students |          0 | PRIMARY  |            1 | StuID       | A         |          25 |     NULL | NULL   |      | BTREE      |         |               |
| students |          1 | idx_age  |            1 | Age         | A         |          25 |     NULL | NULL   |      | BTREE      |         |               |
+----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)

```

```
MariaDB [hellodb]> explain select * from students where age=20 ;
+------+-------------+----------+------+---------------+---------+---------+-------+------+-------+
| id   | select_type | table    | type | possible_keys | key     | key_len | ref   | rows | Extra |
+------+-------------+----------+------+---------------+---------+---------+-------+------+-------+
|    1 | SIMPLE      | students | ref  | idx_age       | idx_age | 1       | const |    2 |       |
+------+-------------+----------+------+---------------+---------+---------+-------+------+-------+
1 row in set (0.00 sec)


```

```
MariaDB [hellodb]> explain select * from students where name like 's%';
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
| id   | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
|    1 | SIMPLE      | students | ALL  | NULL          | NULL | NULL    | NULL |   25 | Using where |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)

```

## 1.1.4 创建name索引

```
MariaDB [hellodb]> create index idx_name on students(name(10));
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0


MariaDB [hellodb]> explain select * from students where name like 's%';
+------+-------------+----------+-------+---------------+----------+---------+------+------+-------------+
| id   | select_type | table    | type  | possible_keys | key      | key_len | ref  | rows | Extra       |
+------+-------------+----------+-------+---------------+----------+---------+------+------+-------------+
|    1 | SIMPLE      | students | range | idx_name      | idx_name | 32      | NULL |    4 | Using where |
+------+-------------+----------+-------+---------------+----------+---------+------+------+-------------+
1 row in set (0.01 sec)



```

## 1.1.5 删除name，age索引

```
MariaDB [hellodb]> drop index idx_age on students;
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [hellodb]> drop index idx_name on students;
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0


```

## 1.1.6 创建复合索引

```
MariaDB [hellodb]> create index idx_name_age on students(name,age);
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [hellodb]> show indexes from students\G
*************************** 1. row ***************************
        Table: students
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: StuID
    Collation: A
  Cardinality: 25
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
*************************** 2. row ***************************
        Table: students
   Non_unique: 1
     Key_name: idx_name_age
 Seq_in_index: 1
  Column_name: Name
    Collation: A
  Cardinality: 25
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
*************************** 3. row ***************************
        Table: students
   Non_unique: 1
     Key_name: idx_name_age
 Seq_in_index: 2
  Column_name: Age
    Collation: A
  Cardinality: 25
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
3 rows in set (0.00 sec)

```

```
MariaDB [hellodb]> explain select * from students where name like 's%';
+------+-------------+----------+-------+---------------+--------------+---------+------+------+-----------------------+
| id   | select_type | table    | type  | possible_keys | key          | key_len | ref  | rows | Extra                 |
+------+-------------+----------+-------+---------------+--------------+---------+------+------+-----------------------+
|    1 | SIMPLE      | students | range | idx_name_age  | idx_name_age | 152     | NULL |    4 | Using index condition |
+------+-------------+----------+-------+---------------+--------------+---------+------+------+-----------------------+
1 row in set (0.00 sec)

MariaDB [hellodb]> explain select * from students where age =20;
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
| id   | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
|    1 | SIMPLE      | students | ALL  | NULL          | NULL | NULL    | NULL |   25 | Using where |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.01 sec)

MariaDB [hellodb]> explain select * from students where name like '%s%';
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
| id   | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
|    1 | SIMPLE      | students | ALL  | NULL          | NULL | NULL    | NULL |   25 | Using where |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)


MariaDB [hellodb]> explain select * from students where name like 's%' and age=20;

MariaDB [hellodb]> explain select * from students where stuid >10;
+------+-------------+----------+-------+---------------+---------+---------+------+------+-------------+
| id   | select_type | table    | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+------+-------------+----------+-------+---------------+---------+---------+------+------+-------------+
|    1 | SIMPLE      | students | range | PRIMARY       | PRIMARY | 4       | NULL |   15 | Using where |
+------+-------------+----------+-------+---------------+---------+---------+------+------+-------------+
1 row in set (0.00 sec)

MariaDB [hellodb]> explain select * from students where stuid +10> 20;
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
| id   | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
|    1 | SIMPLE      | students | ALL  | NULL          | NULL | NULL    | NULL |   25 | Using where |
+------+-------------+----------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)
 
```

## 1.1.7 配置文件加入userstat选项

```
userstat

MariaDB [hellodb]> show variables like 'userstat';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| userstat      | ON    |
+---------------+-------+
1 row in set (0.00 sec)

MariaDB [hellodb]> 
MariaDB [hellodb]> show index_statistics;
Empty set (0.00 sec)

MariaDB [hellodb]> select * from students where stuid=20;
+-------+-----------+-----+--------+---------+-----------+
| StuID | Name      | Age | Gender | ClassID | TeacherID |
+-------+-----------+-----+--------+---------+-----------+
|    20 | Diao Chan |  19 | F      |       7 |      NULL |
+-------+-----------+-----+--------+---------+-----------+
1 row in set (0.00 sec)

MariaDB [hellodb]> show index_statistics;
+--------------+------------+------------+-----------+
| Table_schema | Table_name | Index_name | Rows_read |
+--------------+------------+------------+-----------+
| hellodb      | students   | PRIMARY    |         2 |
+--------------+------------+------------+-----------+
1 row in set (0.00 sec)

```

## 1.1.8 导入脚本

```
[root@node1 ~]# mysql hellodb< testlog.sql 
MariaDB [hellodb]> select * from testlog;
Empty set (0.00 sec)

MariaDB [hellodb]> call pro_testlog;
MariaDB [hellodb]> select * from testlog where name='wang99999';
MariaDB [hellodb]> explain select * from testlog where name='wang99999';
MariaDB [hellodb]> select * from testlog where name='wang99999';

```

## 1.1.9 创建唯一键

```
MariaDB [hellodb]> create unique index uni_age on testlog(age);
MariaDB [hellodb]> show indexes from testlog;

```

## 2.1.1 加入读锁

```
MariaDB [hellodb]> lock tables students read;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> select * from students limit 3;
+-------+-------------+-----+--------+---------+-----------+
| StuID | Name        | Age | Gender | ClassID | TeacherID |
+-------+-------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu |  22 | M      |       2 |         3 |
|     2 | Shi Potian  |  22 | M      |       1 |         7 |
|     3 | Xie Yanke   |  53 | M      |       2 |        16 |
+-------+-------------+-----+--------+---------+-----------+
3 rows in set (0.00 sec)

MariaDB [hellodb]> update students set classid=1 where stuid=1;
ERROR 1099 (HY000): Table 'students' was locked with a READ lock and can't be updated
MariaDB [hellodb]> 

```

## 2.1.2 查看mysql进程

```
MariaDB [hellodb]> show processlist;
MariaDB [hellodb]> kill 6;

```

## 2.1.3 加上写锁

```
MariaDB [hellodb]> lock tables students write;
MariaDB [hellodb]> update students set classid=1 where stuid=3;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [hellodb]> select * from students limit 3;
+-------+-------------+-----+--------+---------+-----------+
| StuID | Name        | Age | Gender | ClassID | TeacherID |
+-------+-------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu |  22 | M      |       1 |         3 |
|     2 | Shi Potian  |  22 | M      |       1 |         7 |
|     3 | Xie Yanke   |  53 | M      |       1 |        16 |
+-------+-------------+-----+--------+---------+-----------+
3 rows in set (0.00 sec)


```

## 2.1.4  另外一个会话连接

```
MariaDB [hellodb]> update students set classid=1 where stuid=1;
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    10
Current database: hellodb

ERROR 2013 (HY000): Lost connection to MySQL server during query
MariaDB [hellodb]> select * from students;
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    11
Current database: hellodb

```

## 2.1.5 解锁

```
MariaDB [hellodb]> unlock tables ;
Query OK, 0 rows affected (0.00 sec)



```

## 2.1.6 备份前加全局读锁

```
MariaDB [hellodb]> flush tables with read lock;
Query OK, 0 rows affected (0.00 sec)

```

## 2.1.7 修改事务是否自动提交

```
autocommit=OFF

MariaDB [hellodb]> delete from students where stuid=25;
MariaDB [hellodb]> commit;
```

## 2.1.8  开启一个事务

```
MariaDB [hellodb]> begin;
MariaDB [hellodb]> insert students (name,age,gender) values('a',30,'f');
MariaDB [hellodb]> insert students (name,age,gender) values('b',33,'f');
MariaDB [hellodb]> select * from students;

|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|    26 | a             |  30 | F      |    NULL |      NULL |
|    27 | b             |  33 | F      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+

MariaDB [hellodb]> rollback;
MariaDB [hellodb]> select * from students;
    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|   24 | Xu Xian       |  27 | M      |    NULL |      NULL |

```

## 2.1.9 提交之后不能撤销

```
MariaDB [hellodb]> begin;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> insert students (name,age,gender) values('b',33,'f');
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> insert students (name,age,gender) values('a',30,'f');
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> commit;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> rollback;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> select * from students;

     24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|    28 | b             |  33 | F      |    NULL |      NULL |
|    29 | a             |  30 | F      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+

```

## 2.2.1 事务提交

```
MariaDB [hellodb]> drop index uni_age on testlog;
MariaDB [hellodb]> begin;
Query OK, 0 rows affected (0.12 sec)

MariaDB [hellodb]> call pro_testlog;
Query OK, 1 row affected (1.14 sec)

MariaDB [hellodb]> commit;
Query OK, 0 rows affected (0.10 sec)

MariaDB [hellodb]> 

```



## 2.2.2 DDM 语言不能撤销

```
MariaDB [hellodb]> begin;
MariaDB [hellodb]> drop table toc;
MariaDB [hellodb]> rollback;
MariaDB [hellodb]> commit;


```

## 2.2.3 支持事务保持点

```
MariaDB [hellodb]> begin;
MariaDB [hellodb]> insert students (name,age,gender) values ('c',23,'f');
MariaDB [hellodb]> savepoint c_tran;
MariaDB [hellodb]> insert students (name,age,gender) values ('d',22,'m');
MariaDB [hellodb]> savepoint d_tran;
MariaDB [hellodb]> insert students (name,age,gender) values ('e',17,'m');
MariaDB [hellodb]> rollback to d_tran;
MariaDB [hellodb]> select * from students;
MariaDB [hellodb]> rollback to c_tran;
MariaDB [hellodb]> rollback;
MariaDB [hellodb]> select * from students;
MariaDB [hellodb]> commit;

```

## 2.2.4 修改隔离级别

```
transaction-isolation=read-uncommitted

MariaDB [(none)]> select @@tx_isolation;
+------------------+
| @@tx_isolation   |
+------------------+
| READ-UNCOMMITTED |
+------------------+
1 row in set (0.00 sec)

```

## 2.2.5 查看事务日志

```
MariaDB [hellodb]> show variables like '%innodb_log%';
+---------------------------+---------+
| Variable_name             | Value   |
+---------------------------+---------+
| innodb_log_block_size     | 512     |
| innodb_log_buffer_size    | 8388608 |
| innodb_log_file_size      | 5242880 |
| innodb_log_files_in_group | 2       |
| innodb_log_group_home_dir | ./      |
+---------------------------+---------+
5 rows in set (0.00 sec)

```

##  2.2.6 事物日志

```
MariaDB [hellodb]> show variables like "%innodb_flush_log_at_trx_commit%";
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| innodb_flush_log_at_trx_commit | 1     |
+--------------------------------+-------+
1 row in set (0.00 sec)

```

## 2.2.7 错误日志

```
MariaDB [hellodb]> show variables like "%log_error%";
+---------------+------------------------------+
| Variable_name | Value                        |
+---------------+------------------------------+
| log_error     | /var/log/mariadb/mariadb.log |
+---------------+------------------------------+
1 row in set (0.00 sec)

```

## 2.2.8 通用日志

```
general-log

MariaDB [hellodb]> show variables like "general%";
+------------------+-----------+
| Variable_name    | Value     |
+------------------+-----------+
| general_log      | ON        |
| general_log_file | node1.log |
+------------------+-----------+
2 rows in set (0.00 sec)


[root@node1 ~]# tail /var/lib/mysql/node1.log -f
		    2 Query	show tables
		    2 Field List	classes 
		    2 Field List	coc 
		    2 Field List	courses 
		    2 Field List	scores 
		    2 Field List	students 
		    2 Field List	teachers 
		    2 Field List	toc 
		    2 Query	select @@version_comment limit 1
201123 22:02:31	    2 Query	show variables like "general%"
201123 22:04:41	    2 Query	select * from students

```

## 2.2.9 通用日志存放的格式

```
MariaDB [hellodb]> show variables like "log_output";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | FILE  |
+---------------+-------+
1 row in set (0.00 sec)


加入/etc/my.cnf
log-output=table

MariaDB [mysql]> select * from general_log;

```

## 2.3.1 慢查询日志

```
MariaDB [mysql]> show variables like "long%";
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)

MariaDB [mysql]> show variables like 'slow_query_log';
+----------------+-------+
| Variable_name  | Value |
+----------------+-------+
| slow_query_log | OFF   |
+----------------+-------+
1 row in set (0.00 sec)

加入/etc/my.cnf
slow-query-log

MariaDB [mysql]> show variables like 'slow_query_log';
+----------------+-------+
| Variable_name  | Value |
+----------------+-------+
| slow_query_log | ON    |
+----------------+-------+
1 row in set (0.00 sec)

加入
long-query-time=3

MariaDB [mysql]> show variables like "long_query_time";
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 3.000000 |
+-----------------+----------+
1 row in set (0.00 sec)


```

## 2.3.2 查看慢查询日志文件

```
先禁用通用日志
MariaDB [hellodb]> select sleep(5);
+----------+
| sleep(5) |
+----------+
|        0 |
+----------+
1 row in set (5.00 sec)

[root@node1 ~]# tail -f /var/lib/mysql/node1-slow.log 

```

## 2.3.3 不使用索引的是否记录日志

```
加入/etc/my.cnf
log-queries-not-using-indexes=on

MariaDB [mysql]> show variables like "log_queries_not_using_indexes";
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| log_queries_not_using_indexes | ON    |
+-------------------------------+-------+
1 row in set (0.00 sec)

MariaDB [hellodb]> select * from students where age=20;

[root@node1 ~]# !tail
tail -f /var/lib/mysql/node1-slow.log 
/usr/libexec/mysqld, Version: 5.5.68-MariaDB (MariaDB Server). started with:
Tcp port: 0  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
# Time: 201123 22:42:11
# User@Host: root[root] @ localhost []
# Thread_id: 2  Schema: hellodb  QC_hit: No
# Query_time: 0.000192  Lock_time: 0.000049  Rows_sent: 2  Rows_examined: 27
use hellodb;
SET timestamp=1606189331;
select * from students where age=20;

```

## 2.3.4 查询慢的过程

```
MariaDB [hellodb]> show variables like 'profiling';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

MariaDB [hellodb]> set profiling=ON;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> show variables like 'profiling';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | ON    |
+---------------+-------+
1 row in set (0.00 sec)

MariaDB [hellodb]> select sleep(1) from teachers;
+----------+
| sleep(1) |
+----------+
|        0 |
|        0 |
|        0 |
|        0 |
+----------+
4 rows in set (4.01 sec)

MariaDB [hellodb]> show profiles;
+----------+------------+---------------------------------+
| Query_ID | Duration   | Query                           |
+----------+------------+---------------------------------+
|        1 | 0.00025730 | show variables like 'profiling' |
|        2 | 4.00687816 | select sleep(1) from teachers   |
+----------+------------+---------------------------------+
2 rows in set (0.00 sec)


MariaDB [hellodb]> show profile for query 2;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000023 |
| Waiting for query cache lock   | 0.000004 |
| checking query cache for query | 0.000032 |
| checking permissions           | 0.000004 |
| Opening tables                 | 0.000009 |
| After opening tables           | 0.000003 |
| System lock                    | 0.000002 |
| Table lock                     | 0.000001 |
| After table lock               | 0.000003 |
| init                           | 0.000007 |
| optimizing                     | 0.000005 |
| statistics                     | 0.000011 |
| preparing                      | 0.000007 |
| executing                      | 0.000002 |
| Sending data                   | 0.000017 |
| User sleep                     | 1.003495 |
| User sleep                     | 1.002361 |
| User sleep                     | 1.000619 |
| User sleep                     | 1.000190 |
| end                            | 0.000012 |
| query end                      | 0.000009 |
| closing tables                 | 0.000011 |
| freeing items                  | 0.000003 |
| updating status                | 0.000014 |
| logging slow query             | 0.000033 |
| cleaning up                    | 0.000002 |
+--------------------------------+----------+
26 rows in set (0.00 sec)


```

## 2.3.5 整理表optimize

```
[root@node1 ~]# mysql hellodb < testlog.sql 
ERROR 1304 (42000) at line 5: PROCEDURE pro_testlog already exists
[root@node1 ~]# 
[root@node1 ~]# mysql hellodb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [hellodb]> call pro_testlog;
Query OK, 1 row affected (0.91 sec)

MariaDB [hellodb]> call pro_testlog;
Query OK, 1 row affected (0.89 sec)

MariaDB [hellodb]> delete from testlog;
Query OK, 199998 rows affected (0.30 sec)

MariaDB [hellodb]> optimize table testlog;
+-----------------+----------+----------+-------------------------------------------------------------------+
| Table           | Op       | Msg_type | Msg_text                                                          |
+-----------------+----------+----------+-------------------------------------------------------------------+
| hellodb.testlog | optimize | note     | Table does not support optimize, doing recreate + analyze instead |
| hellodb.testlog | optimize | status   | OK                                                                |
+-----------------+----------+----------+-------------------------------------------------------------------+
2 rows in set (0.07 sec)

MariaDB [hellodb]> 

```

```
[root@node1 ~]# ll -h /var/lib/mysql/hellodb/testlog.ibd 
-rw-rw---- 1 mysql mysql 96K Nov 23 22:59 /var/lib/mysql/hellodb/testlog.ibd
[root@node1 ~]# ll -h /var/lib/mysql/hellodb/testlog.ibd 
-rw-rw---- 1 mysql mysql 12M Nov 23 22:59 /var/lib/mysql/hellodb/testlog.ibd
[root@node1 ~]# ll -h /var/lib/mysql/hellodb/testlog.ibd 
-rw-rw---- 1 mysql mysql 16M Nov 23 23:00 /var/lib/mysql/hellodb/testlog.ibd
[root@node1 ~]# ll -h /var/lib/mysql/hellodb/testlog.ibd 
-rw-rw---- 1 mysql mysql 16M Nov 23 23:00 /var/lib/mysql/hellodb/testlog.ibd
[root@node1 ~]# ll -h /var/lib/mysql/hellodb/testlog.ibd 
-rw-rw---- 1 mysql mysql 96K Nov 23 23:01 /var/lib/mysql/hellodb/testlog.ibd
[root@node1 ~]# 

```

## 2.3.6 客户端操作日志

```
[root@node1 ~]# cat .mysql_history

```

## 2.3.7 二进制日志

```
MariaDB [hellodb]> show variables like 'sql_log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| sql_log_bin   | ON    |
+---------------+-------+
1 row in set (0.00 sec)

MariaDB [hellodb]> 
MariaDB [hellodb]> 
MariaDB [hellodb]> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
1 row in set (0.00 sec)


```

## 2.4.8 单独存放二级制日志

```
[root@node1 ~]# vim /etc/my.cnf
[root@node1 ~]# mkdir -p /data/logbin
[root@node1 ~]# chown mysql.mysql /data/logbin
[root@node1 ~]# !sy
systemctl restart mariadb
[root@node1 ~]# ll /data/logbin/
total 8
-rw-rw---- 1 mysql mysql 245 Nov 24 00:52 mysql-bin.000001
-rw-rw---- 1 mysql mysql  30 Nov 24 00:52 mysql-bin.index
[root@node1 ~]# grep log-bin /etc/my.cnf
log-bin=/data/logbin/mysql-bin

```

## 2.4.9 插入数据后，二进制会变大

```
MariaDB [hellodb]> call pro_testlog;
Query OK, 1 row affected (0.89 sec)

MariaDB [hellodb]> commit;
Query OK, 0 rows affected (0.03 sec)


```

```
[root@node1 ~]# ll /data/logbin/ -h
total 65M
-rw-rw---- 1 mysql mysql 37M Nov 24 00:55 mysql-bin.000001
-rw-rw---- 1 mysql mysql  30 Nov 24 00:52 mysql-bin.index

```

## 2.5.1 临时禁用二进制日志功能

```
MariaDB [hellodb]> set sql_log_bin=OFF;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> call pro_testlog;
Query OK, 1 row affected (0.81 sec)

MariaDB [hellodb]> commit;
Query OK, 0 rows affected (0.00 sec)


```

```
[root@node1 logbin]# ll /data/logbin/ -h
total 37M
-rw-rw---- 1 mysql mysql 37M Nov 24 00:55 mysql-bin.000001
-rw-rw---- 1 mysql mysql  30 Nov 24 00:52 mysql-bin.index

```

## 2.5.2 二级制日志默认记录格式是语句

```
MariaDB [hellodb]> show variables like 'binlog_format';
+---------------+-----------+
| Variable_name | Value     |
+---------------+-----------+
| binlog_format | STATEMENT |
+---------------+-----------+
1 row in set (0.00 sec)

```

## 2.5.3 修改二级制日志记录格式为行格式

```
[root@node1 logbin]# grep row /etc/my.cnf
binlog-format=row

[root@node1 logbin]# ll /data/logbin/
total 56488
-rw-rw---- 1 mysql mysql 57833428 Nov 24 01:08 mysql-bin.000001
-rw-rw---- 1 mysql mysql      245 Nov 24 01:08 mysql-bin.000002
-rw-rw---- 1 mysql mysql       60 Nov 24 01:08 mysql-bin.index
[root@node1 logbin]# 
[root@node1 logbin]# 

```

```
MariaDB [hellodb]> call pro_testlog;
Query OK, 1 row affected (0.94 sec)

MariaDB [hellodb]> commit;
Query OK, 0 rows affected (0.01 sec)

MariaDB [hellodb]> delete from testlog;
Query OK, 199998 rows affected (0.34 sec)

MariaDB [hellodb]> commit;
Query OK, 0 rows affected (0.01 sec)


```

```
[root@node1 logbin]# ll /data/logbin/ 
total 72868
-rw-rw---- 1 mysql mysql 57833428 Nov 24 01:08 mysql-bin.000001
-rw-rw---- 1 mysql mysql 10089136 Nov 24 01:11 mysql-bin.000002
-rw-rw---- 1 mysql mysql       60 Nov 24 01:08 mysql-bin.index
[root@node1 logbin]# ll /data/logbin/ 
total 72868
-rw-rw---- 1 mysql mysql 57833428 Nov 24 01:08 mysql-bin.000001
-rw-rw---- 1 mysql mysql 13975845 Nov 24 01:12 mysql-bin.000002
-rw-rw---- 1 mysql mysql       60 Nov 24 01:08 mysql-bin.index
[root@node1 logbin]# 

```

## 2.5.4 二进制日志的最大体积 1G

```
MariaDB [hellodb]> show variables like "max_binlog_size";
+-----------------+------------+
| Variable_name   | Value      |
+-----------------+------------+
| max_binlog_size | 1073741824 |
+-----------------+------------+
1 row in set (0.00 sec)

```

## 2.5.5 查看二级制日志大小

```
MariaDB [hellodb]> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |  57833428 |
| mysql-bin.000002 |  13975845 |
+------------------+-----------+
2 rows in set (0.00 sec)

MariaDB [hellodb]> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |  57833428 |
| mysql-bin.000002 |  13975845 |
+------------------+-----------+
2 rows in set (0.00 sec)


```

## 2.5.6 当前使用的二级制的位置 和大小

```
MariaDB [hellodb]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000002 | 13975845 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)


```

## 2.5.7  文本显示二进制日志

```
MariaDB [hellodb]> show binlog events in 'mysql-bin.000002';

```

## 2.5.8 插入一条数据，查看二进制日志

```
MariaDB [hellodb]> insert students (name,age) values ('mage',30);
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |  57833428 |
| mysql-bin.000002 |  13975845 |
+------------------+-----------+
2 rows in set (0.00 sec)

MariaDB [hellodb]> commit;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |  57833428 |
| mysql-bin.000002 |  13976043 |
+------------------+-----------+
2 rows in set (0.00 sec)

MariaDB [hellodb]> show binlog events in 'mysql-bin.000002' from 13975845;
MariaDB [hellodb]> insert students (name,age) values ('wang',30);
MariaDB [hellodb]> commit;
MariaDB [hellodb]> show binlog events in 'mysql-bin.000002' from 13975845;
MariaDB [hellodb]> delete from students where stuid=27;
MariaDB [hellodb]> commit;
MariaDB [hellodb]> show binlog events in 'mysql-bin.000002' from 13975845;
MariaDB [hellodb]> show binlog events in 'mysql-bin.000002' from 13975845 limit 2,3;

```

## 2.5.9 使用mysqlbinlog客户端工具查看二进制内容

```
[root@node1 logbin]# mysqlbinlog --start-position=13975845 mysql-bin.000002 -v 
```

## 2.6.1 清空teachers表

```
MariaDB [hellodb]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000002 | 13976436 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

MariaDB [hellodb]> delete from teachers;
Query OK, 4 rows affected (0.00 sec)

MariaDB [hellodb]> commit;
Query OK, 0 rows affected (0.00 sec)


```

```
[root@node1 logbin]# mysqlbinlog --start-position=13976436 mysql-bin.000002 -v 

```

## 2.6.2 修改二级制存储格式为语句删除students表

```
MariaDB [hellodb]> show master status;
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    2
Current database: hellodb

+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000003 |      245 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

MariaDB [hellodb]> delete from students;
Query OK, 28 rows affected (0.00 sec)

MariaDB [hellodb]> commit;
Query OK, 0 rows affected (0.00 sec)


```

```
[root@node1 logbin]# mysqlbinlog mysql-bin.000003 -v 
.....
use `hellodb`/*!*/;
SET TIMESTAMP=1606200462/*!*/;
delete from students
/*!*/;
# at 402
#201124  1:47:45 serv
.......
```

## 2.6.3 导出二进制日志形成sql文件



```
[root@node1 logbin]# mysqlbinlog mysql-bin.000003 -v > /data/test.sql

```













