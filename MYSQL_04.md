# MYSQL_04

## 1.1.1 刷新日志

```
flush logs;
```

## 1.1.2 删除二进制日志-0003之前的

```
MariaDB [(none)]> purge binary logs to 'mysql-bin.000003';
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000003 |       472 |
| mysql-bin.000004 |       288 |
| mysql-bin.000005 |       245 |
+------------------+-----------+
3 rows in set (0.00 sec)


```

## 1.1.3 重设二进制日志

```
MariaDB [(none)]> reset master;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       245 |
+------------------+-----------+
1 row in set (0.00 sec)
 
```

## 1.1.4 冷备份 （关闭数据库）

```
主库：node1
关闭数据库
systemctl stop mariadb

备份数据目录
tar Jcvf all.bak.tar.xz /var/lib/mysql/

备份二进制目录
tar Jcvf logbin.tar.xz /data/logbin/

拷贝数据目录 二进制目录 配置文件
scp all.bak.tar.xz logbin.tar.xz /etc/my.cnf 192.168.0.112:/root

备库还原步骤：node2
准备数据库
yum -y install mariadb-server

覆盖配置文件
\cp my.cnf /etc/my.cnf

数据目录
mv var/lib/mysql/* /var/lib/mysql/

二进制目录
mkdir -p /data/logbin/
chown mysql.mysql /data/logbin/
\mv data/logbin/* /data/logbin/

启动mysql
 systemctl start mariadb

```

## 1.1.5 mysqldump 备份单个数据库

```
[root@node1 ~]# mysqldump hellodb > /data/hellodb.sql
[root@node1 ~]# mysql -e 'DROP database hellodb'
[root@node1 ~]# 
[root@node1 ~]# mysql -e 'CREATE DATABASE hellodb'
[root@node1 ~]# mysql hellodb < /data/hellodb.sql 
[root@node1 ~]# 

```

## 1.1.6 -B 备份数据库

```
[root@node1 ~]# mysqldump -B hellodb > /data/hellodb2.sql
[root@node1 ~]# mysql -e 'DROP database hellodb'
[root@node1 ~]# mysql -e 'show databases'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
[root@node1 ~]# mysql < /data/hellodb2.sql 
[root@node1 ~]# mysql -e 'show databases'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hellodb            |
| mysql              |
| performance_schema |
| test               |
+--------------------+
[root@node1 ~]# 

```

## 1.1.7 -A 备份数据库

```
+
MariaDB [hellodb]> CREATE FUNCTION simpleFun() RETURNS VARCHAR(20) RETURN "hello world";
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> quit
Bye
[root@node1 ~]# mysqldump -A > /data/all.sql
[root@node1 ~]# grep simpleFun /data/all.sql

```

## 1.1.8 备份数据库默认字符集是utf8

```
[root@node1 ~]# mysqldump --help|grep utf8
default-character-set             utf8

```

## 1.1.9  备份还原

```
[root@node1 ~]# mysqldump -A --master-data=2 > /data/all.sql 
[root@node1 ~]# mysql -e 'show master logs'
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       245 |
+------------------+-----------+

MariaDB [hellodb]> insert students (name,age,gender) values ('gouzi',18,'M');
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       494 |
+------------------+-----------+
1 row in set (0.00 sec)

MariaDB [hellodb]> insert students (name,age,gender) values ('maozi',18,'M');
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       743 |
+------------------+-----------+
1 row in set (0.00 sec)


```

## 1.2.1 删除数据库模拟故障

```
[root@node1 ~]# rm -rf /var/lib/mysql/*
[root@node1 ~]# systemctl restart mariadb
MariaDB [hellodb]> set sql_log_bin=OFF;
MariaDB [(none)]> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       762 |
| mysql-bin.000002 |     30331 |
| mysql-bin.000003 |   1038814 |
| mysql-bin.000004 |       245 |
+------------------+-----------+
4 rows in set (0.00 sec)

root@node1 ~]# cd /data/logbin/
[root@node1 logbin]# mysqlbinlog mysql-bin.000001 > /data/inc.sql
[root@node1 logbin]# mysqlbinlog mysql-bin.000002 >> /data/inc.sql
[root@node1 logbin]# mysqlbinlog mysql-bin.000003 >> /data/inc.sql
[root@node1 logbin]#
MariaDB [(none)]> source /data/all.sql
MariaDB [(none)]> source /data/inc.sql
MariaDB [(none)]> set sql_log_bin=ON;
MariaDB [hellodb]> select * from students order by stuid desc limit 2;
+-------+-------+-----+--------+---------+-----------+
| StuID | Name  | Age | Gender | ClassID | TeacherID |
+-------+-------+-----+--------+---------+-----------+
|    27 | maozi |  18 | M      |    NULL |      NULL |
|    26 | gouzi |  18 | M      |    NULL |      NULL |
+-------+-------+-----+--------+---------+-----------+
2 rows in set (0.00 sec)


```

## 1.2.1 删除表故障模拟

```
[root@node1 logbin]# mysqldump -A --master-data=2 > /data/all_`date +%F`.sql
[root@node1 logbin]# ll /data/
total 2560
-rw-r--r--. 1 root  root   521644 Nov 25 03:12 all_2020-11-25.sql
-rw-r--r--. 1 root  root   521580 Nov 25 00:25 all.sql
-rw-r--r--. 1 root  root  1571667 Nov 25 00:29 inc.sql
drwxr-xr-x. 2 mysql mysql     125 Nov 25 00:26 logbin
[root@node1 logbin]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show variables like 'sql_log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| sql_log_bin   | ON    |
+---------------+-------+
1 row in set (0.00 sec)

MariaDB [(none)]> use hellodb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [hellodb]> insert students (name,age) values ('c',22);
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> insert students (name,age) values ('d',32);
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> drop table students;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> insert teachers (name,age) values ('wang',18);

exit


```

```
[root@node1 logbin]# grep 000004 /data/all_2020-11-25.sql
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=1589717;
[root@node1 logbin]# 

MariaDB [(none)]> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       762 |
| mysql-bin.000002 |     30331 |
| mysql-bin.000003 |   1038814 |
| mysql-bin.000004 |   1590535 |
+------------------+-----------+
4 rows in set (0.00 sec)

MariaDB [(none)]> flush logs;
Query OK, 0 rows affected (0.00 sec)

[root@node1 logbin]# mysqlbinlog --start-position=1589717 /data/logbin/mysql-bin.000004 > /data/inc.sql
[root@node1 logbin]# vim /data/inc.sql
注释DROP那一行

删除数据库重新还原
rm -rf /var/lib/mysql/*
systemctl restart mariadb
mysql
MariaDB [(none)]> set sql_log_bin=OFF;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> source /data/all_2020-11-25.sql
MariaDB [hellodb]> source /data/inc.sql

MariaDB [hellodb]> select * from students order by stuid desc limit 4;
+-------+-------+-----+--------+---------+-----------+
| StuID | Name  | Age | Gender | ClassID | TeacherID |
+-------+-------+-----+--------+---------+-----------+
|    29 | d     |  32 | F      |    NULL |      NULL |
|    28 | c     |  22 | F      |    NULL |      NULL |
|    27 | maozi |  18 | M      |    NULL |      NULL |
|    26 | gouzi |  18 | M      |    NULL |      NULL |
+-------+-------+-----+--------+---------+-----------+
4 rows in set (0.00 sec)

MariaDB [hellodb]> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   5 | wang          |  18 | NULL   |
+-----+---------------+-----+--------+
5 rows in set (0.00 sec)



```

## 1.2.3 mysqldump -F 滚动多次日志

```
[root@node1 logbin]# ll /data/logbin/
total 3668
-rw-rw----. 1 mysql mysql     762 Nov 25 00:26 mysql-bin.000001
-rw-rw----. 1 mysql mysql   30331 Nov 25 00:26 mysql-bin.000002
-rw-rw----. 1 mysql mysql 1038814 Nov 25 00:26 mysql-bin.000003
-rw-rw----. 1 mysql mysql 1590578 Nov 25 03:20 mysql-bin.000004
-rw-rw----. 1 mysql mysql     264 Nov 25 03:33 mysql-bin.000005
-rw-rw----. 1 mysql mysql   30331 Nov 25 03:33 mysql-bin.000006
-rw-rw----. 1 mysql mysql 1038814 Nov 25 03:33 mysql-bin.000007
-rw-rw----. 1 mysql mysql     245 Nov 25 03:33 mysql-bin.000008
-rw-rw----. 1 mysql mysql     240 Nov 25 03:33 mysql-bin.index
[root@node1 logbin]# mysqldump -A -F > /dev/null
[root@node1 logbin]# ll /data/logbin/
total 3680
-rw-rw----. 1 mysql mysql     762 Nov 25 00:26 mysql-bin.000001
-rw-rw----. 1 mysql mysql   30331 Nov 25 00:26 mysql-bin.000002
-rw-rw----. 1 mysql mysql 1038814 Nov 25 00:26 mysql-bin.000003
-rw-rw----. 1 mysql mysql 1590578 Nov 25 03:20 mysql-bin.000004
-rw-rw----. 1 mysql mysql     264 Nov 25 03:33 mysql-bin.000005
-rw-rw----. 1 mysql mysql   30331 Nov 25 03:33 mysql-bin.000006
-rw-rw----. 1 mysql mysql 1038814 Nov 25 03:33 mysql-bin.000007
-rw-rw----. 1 mysql mysql     288 Nov 25 03:41 mysql-bin.000008
-rw-rw----. 1 mysql mysql     288 Nov 25 03:41 mysql-bin.000009
-rw-rw----. 1 mysql mysql     288 Nov 25 03:41 mysql-bin.000010
-rw-rw----. 1 mysql mysql     245 Nov 25 03:41 mysql-bin.000011
-rw-rw----. 1 mysql mysql     330 Nov 25 03:41 mysql-bin.index
[root@node1 logbin]# 

```

## 1.2.4 滚动一次日志 加-x 

```
[root@node1 logbin]# mysqldump -A -F -x > /dev/null
[root@node1 logbin]# ll /data/logbin/
total 3684
-rw-rw----. 1 mysql mysql     762 Nov 25 00:26 mysql-bin.000001
-rw-rw----. 1 mysql mysql   30331 Nov 25 00:26 mysql-bin.000002
-rw-rw----. 1 mysql mysql 1038814 Nov 25 00:26 mysql-bin.000003
-rw-rw----. 1 mysql mysql 1590578 Nov 25 03:20 mysql-bin.000004
-rw-rw----. 1 mysql mysql     264 Nov 25 03:33 mysql-bin.000005
-rw-rw----. 1 mysql mysql   30331 Nov 25 03:33 mysql-bin.000006
-rw-rw----. 1 mysql mysql 1038814 Nov 25 03:33 mysql-bin.000007
-rw-rw----. 1 mysql mysql     288 Nov 25 03:41 mysql-bin.000008
-rw-rw----. 1 mysql mysql     288 Nov 25 03:41 mysql-bin.000009
-rw-rw----. 1 mysql mysql     288 Nov 25 03:41 mysql-bin.000010
-rw-rw----. 1 mysql mysql     359 Nov 25 03:42 mysql-bin.000011
-rw-rw----. 1 mysql mysql     245 Nov 25 03:42 mysql-bin.000012
-rw-rw----. 1 mysql mysql     360 Nov 25 03:42 mysql-bin.index
[root@node1 logbin]# 

```

## 1.2.5 备份

```
[root@node1 logbin]# mysqldump -F -A --single-transaction --master-data=2 > /data/all.sql

```

## 1.2.6 分库备份

```
[root@node1 ~]# for db in `mysql -e 'show databases'|grep -vE '^(information_schema|performance_schema|Database)$'`;do mysqldump -B $db --single-transaction --master-data=2|gzip > /data/$db.sql.gz;done

[root@node1 ~]# ls /data -lt
total 1180
-rw-r--r--. 1 root  root     605 Nov 25 05:05 test.sql.gz
-rw-r--r--. 1 root  root  139589 Nov 25 05:05 mysql.sql.gz
-rw-r--r--. 1 root  root    2020 Nov 25 05:05 hellodb.sql.gz
-rw-r--r--. 1 root  root  521711 Nov 25 03:51 all.sql
drwxr-xr-x. 2 mysql mysql   4096 Nov 25 03:51 logbin
-rw-r--r--. 1 root  root    3215 Nov 25 03:26 inc.sql
-rw-r--r--. 1 root  root  521644 Nov 25 03:12 all_2020-11-25.sql
[root@node1 ~]# 

```

```
[root@node1 ~]# mysql -e 'show databases'|grep -vE '^(information_schema|performance_schema|Database)$'|sed -rn 's@(.*)@mysqldump -B \1 --single-transaction --master-data=2 |gzip > /data/\1\.sql\.gz@p'|bash

```

## 1.2.7 安装 percona-xtrabackup

```
yum -y install epel*
yum -y install percona-xtrabackup-24-2.4.7-2.el7.x86_64.rpm 

[root@node1 ~]# rpm -ql percona-xtrabackup-24
/usr/bin/innobackupex
/usr/bin/xbcloud
/usr/bin/xbcloud_osenv
/usr/bin/xbcrypt
/usr/bin/xbstream
/usr/bin/xtrabackup
/usr/share/doc/percona-xtrabackup-24-2.4.7
/usr/share/doc/percona-xtrabackup-24-2.4.7/COPYING
/usr/share/man/man1/innobackupex.1.gz
/usr/share/man/man1/xbcrypt.1.gz
/usr/share/man/man1/xbstream.1.gz
/usr/share/man/man1/xtrabackup.1.gz
[root@node1 ~]# ll /usr/bin/innobackupex 
lrwxrwxrwx. 1 root root 10 Nov 25 06:51 /usr/bin/innobackupex -> xtrabackup
[root@node1 ~]# 


```

## 1.2.8 xtrabackup 执行备份

```
[root@node1 ~]# mkdir /backup
[root@node1 ~]# 
[root@node1 ~]# 
[root@node1 ~]# xtrabackup --backup --target-dir=/backup

```

### 1.2.9 备份序列号

```
[root@node1 ~]# cat /backup/xtrabackup_checkpoints 
backup_type = full-backuped
from_lsn = 0
to_lsn = 1597945
last_lsn = 1597945
compact = 0
recover_binlog_info = 0

```

### 1.3.1 拷贝数据目录

```
[root@node1 ~]# scp -r /backup/ 192.168.0.112:/root

```

### 1.3.2 整理数据库

```
[root@node2 ~]# xtrabackup --prepare --target-dir=/root/backup

```

### 1.3.3 复制数据（数据目录为空，服务为停止状态）

```
[root@node2 ~]# xtrabackup --copy-back --target-dir=/root/backup/
[root@node2 ~]# chown -R mysql.mysql /var/lib/mysql/
[root@node2 ~]# systemctl start mariadb
[root@node2 ~]# 

```

## 1.3.4 增量备份机还原

```
[root@node1 ~]# rm -rf /backup/*
先完全备份
[root@node1 ~]# xtrabackup --backup --target-dir=/backup/base
MariaDB [hellodb]> insert teachers (name,age,gender) values ('mage',37,'M');

第一次增量备份
[root@node1 ~]# xtrabackup --backup --target-dir=/backup/inc1 --incremental-basedir=/backup/base

[root@node1 ~]# du -sh /backup/*
20M	/backup/base
3.5M	/backup/inc1

MySQL库增加一条记录
MariaDB [hellodb]> insert teachers (name,age,gender) values ('wang',37,'M');

第二次增量备份
[root@node1 ~]# xtrabackup --backup --target-dir=/backup/inc2 --incremental-basedir=/backup/inc1

还原步骤：node2
[root@node2 ~]# rm -rf backup/
[root@node2 ~]# scp -r 192.168.0.111:/backup/ /

整理数据
[root@node2 ~]# xtrabackup --prepare --apply-log-only --target-dir=/backup/base
[root@node2 ~]# xtrabackup --prepare --apply-log-only --target-dir=/backup/base --incremental-dir=/backup/inc1
[root@node2 ~]# xtrabackup --prepare --target-dir=/backup/base --incremental-dir=/backup/inc2

停止数据库并且清空数据
[root@node2 ~]# systemctl stop mariadb
[root@node2 ~]# ll /var/lib/mysql/
total 0

复制到数据库目录（数据库目录必须为空，不能启动服务）
[root@node2 ~]# systemctl stop mariadb
[root@node2 ~]# ll /var/lib/mysql/
total 0
[root@node2 ~]# xtrabackup --copy-back --target-dir=/backup/base

还原属性
[root@node2 ~]# chown -R mysql.mysql /var/lib/mysql

启动服务
[root@node2 ~]# systemctl start mariadb

查看数据是否恢复完毕
MariaDB [hellodb]> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   5 | mage          |  37 | M      |
|   6 | wang          |  37 | M      |
+-----+---------------+-----+--------+
6 rows in set (0.00 sec)


```

1.3.5 MySQL复制

```
主节点配置
[mysqld]
server-id=1
log-bin=/data/logbin/mysql-bin

重启mysql服务器
systemctl restart mariadb

授权复制用户
mysql
MariaDB [(none)]> grant replication slave on *.* to 'repluser'@'192.168.0.%' identified by 'shaoxh';
MariaDB [(none)]> show master logs;
.......
| mysql-bin.000020 |       401 |

```

```
从节点配置
mysql
MariaDB [(none)]> CHANGE MASTER TO
    -> MASTER_HOST='192.168.0.111',
    -> MASTER_USER='repluser',
    -> MASTER_PASSWORD='shaoxh',
    -> MASTER_PORT=3306,
    -> MASTER_LOG_FILE='mysql-bin.000020',
    -> MASTER_LOG_POS=401;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G
MariaDB [(none)]> show processlist;



```

### 1.3.5 测试主节点创建db1数据库

```
MariaDB [(none)]> create database db1;

从节点查看同步情况
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| hellodb            |
| mysql              |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.00 sec)

```

### 1.3.6 导入testlog.sql 测试

```
[root@node1 ~]# mysql db1
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [db1]> source /root/testlog.sql
Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

MariaDB [db1]> call pro_testlog;
Query OK, 1 row affected (25.88 sec)

MariaDB [db1]> call pro_testlog;
Query OK, 1 row affected (26.54 sec)

MariaDB [db1]> 

```

```
[root@node2 ~]# mysql db1
MariaDB [db1]> select count(*) from testlog;
+----------+
| count(*) |
+----------+
|   114821 |
+----------+
1 row in set (0.02 sec)

MariaDB [db1]> select count(*) from testlog;
+----------+
| count(*) |
+----------+
|   118245 |
+----------+
1 row in set (0.01 sec)

MariaDB [db1]> select count(*) from testlog;
+----------+
| count(*) |
+----------+
|   121124 |
+----------+
1 row in set (0.02 sec)

```



## 1.3.7 主从复制

```
主节点
启用二进制功能
log-bin=mysql-bin

设置全局唯一的ID号
server-id=1

创建有复制权限的用户账号
 GRANT REPLICATION SLAVE ON *.* TO 'repluser'@'192.168.0.%' IDENTIFIED BY 'replpass';
 
查看postion 点位置
MariaDB [(none)]> SHOW MASTER logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |     30331 |
| mysql-bin.000002 |   1038814 |
| mysql-bin.000003 |       403 |
+------------------+-----------+

从节点配置
设置全局唯一的ID号
server-id=2

MariaDB [(none)]> CHANGE MASTER TO
    -> MASTER_HOST='192.168.0.111',
    -> MASTER_USER='repluser',
    -> MASTER_PASSWORD='replpass',
    -> MASTER_PORT=3306,
    -> MASTER_LOG_FILE='mysql-bin.000003',
    -> MASTER_LOG_POS=403;
MariaDB [(none)]> SHOW SLAVE status\G
MariaDB [(none)]> START SLAVE;
MariaDB [(none)]> SHOW PROCESSLIST;

```

















