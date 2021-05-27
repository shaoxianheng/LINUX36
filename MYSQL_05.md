# MYSQL_05

##  1.1.1 同步已经在生产中使用的mysql数据库

```
主节点的数据库列表：
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hellodb            |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)

MariaDB [(none)]> 

配置主服务器的server-id和二进制路径
[root@node1 ~]# vim /etc/my.cnf
[mysqld]  里面加入如下两行
server-id=111
log-bin

重新启动mariadb
[root@node1 ~]# systemctl restart mariadb
MariaDB [(none)]> show master logs;
+--------------------+-----------+
| Log_name           | File_size |
+--------------------+-----------+
| mariadb-bin.000001 |       245 |
+--------------------+-----------+
1 row in set (0.00 sec)

MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO repluser@'192.168.0.%' IDENTIFIED BY 'replpass';
Query OK, 0 rows affected (0.00 sec)

备份所有数据库
[root@node1 ~]# mysqldump -A --single-transaction --master-data=1 -F > all.sql
[root@node1 ~]# scp all.sql 192.168.0.112:/root

从节点操作步骤：
修改一下授权用户和主机
[root@node2 ~]# vim all.sql 
CHANGE MASTER TO
MASTER_HOST='192.168.0.111',
MASTER_USER='repluser',
MASTER_PASSWORD='replpass',
MASTER_PORT=3306,
MASTER_LOG_FILE='mariadb-bin.000002',
MASTER_LOG_POS=245;

[root@node2 ~]# yum -y install mariadb-server

加入server-id
[root@node2 ~]# vim /etc/my.cnf
[mysqld]
server-id=112
read-only

启动数据库，恢复
[root@node2 ~]# systemctl start mariadb
[root@node2 ~]# mysql < all.sql 
MariaDB [(none)]> SHOW SLAVE STATUS\G
MariaDB [(none)]> SHOW VARIABLES LIKE 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 112   |
+---------------+-------+
1 row in set (0.00 sec)

MariaDB [(none)]> SHOW VARIABLES LIKE 'read_only';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| read_only     | ON    |
+---------------+-------+
1 row in set (0.00 sec)

[root@node2 ~]# ss -nt
State       Recv-Q Send-Q                           Local Address:Port                                          Peer Address:Port              
ESTAB       0      0                                192.168.0.112:22                                           192.168.0.104:57622              
[root@node2 ~]# 

MariaDB [(none)]> START SLAVE;
[root@node2 ~]# ss -nt
State       Recv-Q Send-Q                           Local Address:Port                                          Peer Address:Port              
ESTAB       0      0                                192.168.0.112:37992                                        192.168.0.111:3306               
ESTAB       0      52                               192.168.0.112:22                                           192.168.0.104:57622   


MariaDB [(none)]> select user,host,password from mysql.user;
+----------+-------------+-------------------------------------------+
| user     | host        | password                                  |
+----------+-------------+-------------------------------------------+
| root     | localhost   |                                           |
| root     | node1       |                                           |
| root     | 127.0.0.1   |                                           |
| root     | ::1         |                                           |
|          | localhost   |                                           |
|          | node1       |                                           |
| repluser | 192.168.0.% | *D98280F03D0F78162EBDBB9C883FC01395DEA2BF |
+----------+-------------+-------------------------------------------+
7 rows in set (0.00 sec)

主库创建数据库测试
MariaDB [(none)]> create database tb1;
Query OK, 1 row affected (0.01 sec)

从库查看是否创建成功
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hellodb            |
| mysql              |
| performance_schema |
| tb1                |
| test               |
+--------------------+
6 rows in set (0.00 sec)


```

## 1.1.2 从库创建一个库导致不能同步解决方法

```
从库创建
MariaDB [(none)]> CREATE DATABASE DB1;

主库创建
MariaDB [(none)]> CREATE DATABASE DB1;

从库查看同步状态
MariaDB [(none)]> show slave status\G
出现了错误，

解决方法：
从库执行
MariaDB [(none)]> DROP DATABASE DB1;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> stop SLAVE;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> START  SLAVE;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> SHOW SLAVE STATUS\G

```

## 1.1.3 主库-从库表插入数据导致冲突

```
从库执行
MariaDB [hellodb]> INSERT teachers (name,age) VALUES('mage',20);
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> SELECT * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   5 | mage          |  20 | NULL   |
+-----+---------------+-----+--------+
5 rows in set (0.00 sec)

主库执行
MariaDB [hellodb]> INSERT teachers (name,age) VALUES('wang',20);
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> SELECT * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   5 | wang          |  20 | NULL   |
+-----+---------------+-----+--------+
5 rows in set (0.00 sec)

查看从库的同步状态
MariaDB [hellodb]> SHOW SLAVE STATUS\G
出现同步错误

主库执行
如果还有数据插入，
[root@node1 ~]# mysql hellodb <testlog.sql 
MariaDB [hellodb]> call pro_testlog;

从库执行
从库可以跳过一次错误
MariaDB [hellodb]> stop slave;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> set global sql_slave_skip_counter=1;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> start slave;
Query OK, 0 rows affected (0.00 sec)

MariaDB [hellodb]> SHOW SLAVE STATUS\G
MariaDB [hellodb]> show tables;
+-------------------+
| Tables_in_hellodb |
+-------------------+
| classes           |
| coc               |
| courses           |
| scores            |
| students          |
| teachers          |
| testlog           |
| toc               |
+-------------------+
8 rows in set (0.00 sec)
MariaDB [hellodb]> select count(*) from testlog;


```

### 1.1.4 第二种方法跳过错误

```
[root@node2 ~]# vim /etc/my.cnf
slave-skip-errors=1062
[root@node2 ~]# systemctl restart mariadb
[root@node2 ~]# mysql -e "show slave status\G"|grep Yes
             Slave_IO_Running: Yes
             Slave_SQL_Running: Yes

测试是否可以继续同步：
主库操作：
MariaDB [hellodb]> INSERT teachers (name,age) VALUES('wang',20);
MariaDB [hellodb]> SELECT * FROM teachers ORDER BY TID DESC LIMIT 1;
+-----+------+-----+--------+
| TID | Name | Age | Gender |
+-----+------+-----+--------+
|   7 | wang |  20 | NULL   |
+-----+------+-----+--------+
1 row in set (0.00 sec)


从库操作：
MariaDB [hellodb]> SELECT * FROM teachers ORDER BY TID DESC LIMIT 1;
+-----+------+-----+--------+
| TID | Name | Age | Gender |
+-----+------+-----+--------+
|   7 | wang |  20 | NULL   |
+-----+------+-----+--------+
1 row in set (0.00 sec)


```

## 1.1.5  在创建一个从库

```
首先需要备份主库的数据库
[root@node1 ~]# mysqldump -A --single-transaction --master-data=1 -F > all.sql
[root@node1 ~]# scp all.sql 192.168.0.113:/root

从库步骤
安装mariadb数据库
[root@node3 ~]# yum -y install mariadb-server

修改备份的sql脚本
[root@node3 ~]# vim all.sql 
CHANGE MASTER TO
MASTER_HOST='192.168.0.111',
MASTER_USER='repluser',
MASTER_PASSWORD='replpass',
MASTER_PORT=3306,
MASTER_LOG_FILE='mariadb-bin.000003', MASTER_LOG_POS=245;

修改从库配置文件
[root@node3 ~]# cat /etc/my.cnf
[mysqld]
server-id=113
read-only

启动mysql
[root@node3 ~]# systemctl start mariadb

导入数据库文件
[root@node3 ~]# mysql < all.sql 
mysql
MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show slave status \G

主库操作：
建立一个db5是否同步
MariaDB [(none)]> create database db5;

从库操作：查看数据库列表
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| DB1                |
| db5                |
| hellodb            |
| mysql              |
| performance_schema |
| tb1                |
| test               |
+--------------------+





```

## 1.1.6 模拟主库在更新表时关机

```
主库操作：
MariaDB [hellodb]> call pro_testlog;
....
虚拟机关机。

现在提升112 为主库
先停止同步
MariaDB [(none)]> stop slave;
清理同步信息
MariaDB [(none)]> reset slave;
彻底清理
MariaDB [(none)]> reset slave all;
MariaDB [(none)]> show slave status\G

修改配置文件
[root@node2 ~]# cat /etc/my.cnf
[mysqld]
server-id=112
log-bin
#read-only  删除或者注释

重启mysql服务
[root@node2 ~]# systemctl restart mariadb
mysql
MariaDB [(none)]> select user,host,password from mysql.user;
MariaDB [(none)]> show master logs;
+--------------------+-----------+
| Log_name           | File_size |
+--------------------+-----------+
| mariadb-bin.000001 |       245 |
+--------------------+-----------+
1 row in set (0.00 sec)

从库执行：113，node3
MariaDB [(none)]> stop slave;
MariaDB [(none)]> reset slave all;
MariaDB [(none)]> CHANGE MASTER TO
    -> MASTER_HOST='192.168.0.112',
    -> MASTER_USER='repluser',
    -> MASTER_PASSWORD='replpass',
    -> MASTER_PORT=3306,
    -> MASTER_LOG_FILE='mariadb-bin.000001',
    -> MASTER_LOG_POS=245;
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status \G
             Slave_IO_Running: Yes
             Slave_SQL_Running: Yes

112 执行；测试 主库删除db5
MariaDB [(none)]> drop database db5;

113 执行；验证 从库是否还要db5
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| DB1                |
| hellodb            |
| mysql              |
| performance_schema |
| tb1                |
| test               |
+--------------------+
7 rows in set (0.00 sec)


```

## 1.1.7 级联复制

```
环境准备
主111  从112  从113

主111操作：
安装mysql
[root@node1 ~]# yum -y install mariadb-server

修改配置文件
[root@node1 ~]# cat /etc/my.cnf
[mysqld]
server-id=111
log-bin

启动mysql
[root@node1 ~]# systemctl start mariadb
[root@node1 ~]# mysql

用户授权
MariaDB [(none)]> grant replication slave on *.* to repluser@'192.168.0.%' identified by 'replpass';
MariaDB [(none)]> exit

备份数据库
[root@node1 ~]# mysqldump -A --single-transaction --master-data=1 -F > all.sql

拷贝数据库文件
[root@node1 ~]# scp all.sql 192.168.0.112:/root
[root@node1 ~]# scp all.sql 192.168.0.113:/root

从112 操作：
[root@node2 ~]# yum -y install mariadb-server

修改配置文件
[root@node2 ~]# cat /etc/my.cnf
[mysqld]
log_bin
log_slave_updates
server-id=112
read-only

启动mysql
[root@node2 ~]# systemctl start mariadb

修改数据文件
[root@node2 ~]# vim all.sql
CHANGE MASTER TO
MASTER_HOST='192.168.0.111',
MASTER_USER='repluser',
MASTER_PASSWORD='replpass',
MASTER_PORT=3306,
MASTER_LOG_FILE='mariadb-bin.000002', MASTER_LOG_POS=245;

导入数据文件
[root@node2 ~]# mysql < all.sql 
[root@node2 ~]# mysql
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status \G
            Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

测试主库111 创建db1数据库：
MariaDB [(none)]> create database db1;

验证从库112 是否有db1数据库
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)

从库113 安装mysql
[root@node3 ~]# yum -y install mariadb-server

修改配置文件
[root@node3 ~]# cat /etc/my.cnf
[mysqld]
server-id=113
read-only

启动mysql
[root@node3 ~]# systemctl start mariadb

备份级联从库的数据
[root@node2 ~]# mysqldump -A --single-transaction --master-data=1 -F > all.sql
[root@node2 ~]# scp all.sql 192.168.0.113:/root

准备导入数据文件
[root@node3 ~]# vim all.sql 
CHANGE MASTER TO 
MASTER_HOST='192.168.0.112',  ###指定级联从库ip
MASTER_USER='repluser',
MASTER_PASSWORD='replpass',
MASTER_PORT=3306,
MASTER_LOG_FILE='mariadb-bin.000002', MASTER_LOG_POS=245;   

[root@node3 ~]# mysql < all.sql 

从库113 启动复制线程
MariaDB [(none)]> start slave;

测试主库创建shaoheng数据库；
MariaDB [(none)]> create database shaoheng;
Query OK, 1 row affected (0.00 sec)

验证级联从库

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| shaoheng           |
| test               |
+--------------------+
5 rows in set (0.00 sec)

验证从库：
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| shaoheng           |
| test               |
+--------------------+
5 rows in set (0.00 sec)







```

## 1.1.8 主主复制

```

容易产生的问题：数据不一致；因此慎用
考虑要点：自动增长id
主节点111操作：
安装mysql
[root@node1 ~]# yum -y install mariadb-server

修改配置文件
[root@node1 ~]# cat /etc/my.cnf
[mysqld]
server-id=1
log-bin
auto-increment-offset=1
auto-increment-increment=2

创建授权用户
[root@node1 ~]# systemctl start mariadb
[root@node1 ~]# mysql
MariaDB [(none)]> grant replication slave on *.* to repluser@'192.168.0.%' identified by 'replpass';
MariaDB [(none)]> flush privileges;

备份数据库
[root@node1 ~]# mysqldump -A --single-transaction --master-data=1 -F > all.sql
[root@node1 ~]# scp all.sql 192.168.0.112:/root

主节点112操作：
[root@node2 ~]# yum -y install mariadb-server

修改配置文件
[root@node2 ~]# cat /etc/my.cnf
[mysqld]
server-id=2
log-bin
auto-increment-offset=2
auto-increment-crement=2

启动数据库导入数据
[root@node2 ~]# systemctl start mariadb
[root@node2 ~]# vim all.sql
CHANGE MASTER TO
MASTER_HOST='192.168.0.111',
MASTER_USER='repluser',
MASTER_PASSWORD='replpass',
MASTER_PORT=3306,
MASTER_LOG_FILE='mariadb-bin.000002', MASTER_LOG_POS=245;
[root@node2 ~]# mysql < all.sql
[root@node2 ~]# mysql
MariaDB [(none)]> start slave;

主节点111操作：
测试创建db1数据库
MariaDB [(none)]> create database db1;

主节点112操作：
验证创建db1数据库是否同步

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
单项的主从复制完成，双向的复制如下

主节点112操作：
查看postion位置
MariaDB [(none)]> show master logs;
+--------------------+-----------+
| Log_name           | File_size |
+--------------------+-----------+
| mariadb-bin.000001 |    514951 |
+--------------------+-----------+
1 row in set (0.00 sec)

主节点111操作：
连接主库
MariaDB [(none)]> CHANGE MASTER TO MASTER_HOST='192.168.0.112', MASTER_USER='repluser', MASTER_PASSWORD='replpass', MASTER_PORT=3306, MASTER_LOG_
FILE='mariadb-bin.000001', MASTER_LOG_POS=514951;
刷新权限
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G

测试主节点112 创建db2数据库
MariaDB [(none)]> create database db2;

验证 主节点111 查询db2 是否存在
MariaDB [(none)]> show databases;

主节点111 操作：
[root@node1 ~]# mysql < hellodb_innodb.sql 
[root@node1 ~]# mysql hellodb
MariaDB [hellodb]> create table test (id int auto_increment primary key,name char(10));

调出xshll全部加入会话：
insert test (name) values ('wang');
select * from test;
显示结果：
+----+------+
| id | name |
+----+------+
|  1 | wang |
|  2 | wang |
+----+------+

会话执行：
insert test (name) values ('a'),('b');
select * from test;
显示结果：
+----+------+
| id | name |
+----+------+
|  1 | wang |
|  2 | wang |
|  3 | a    |
|  4 | a    |
|  5 | b    |
|  6 | b    |
+----+------+


如果全部会话中执行，就会出现错误
create table test2(id int);
show slave status\G

处理方法：
stop slave;
set global sql_slave_skip_counter=1;
start slave;
show slave status\G

如果是三个主模式：
auto-increment-offset=1 开始点        （1 3 5）
auto-increment-increment=3 增长幅度

auto-increment-offset=2 开始点        （2 5 8）
auto-increment-increment=3 增长幅度

auto-increment-offset=3 开始点        （3 6 9）
auto-increment-increment=3 增长幅度
```

## 1.1.9 半同步复制

```##
环境准备 三台主机 主111， 从112， 从113
主111节点操作：
安装数据库
[root@node1 ~]# yum -y install mariadb-server

修改配置文件
[root@node1 ~]# cat /etc/my.cnf
[mysqld]
server-id=1
log-bin

启动mysql
[root@node1 ~]# systemctl start mariadb
[root@node1 ~]# mysql
MariaDB [(none)]> grant replication slave on *.* to repluser@'192.168.0.%' identified by 'replpass';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;

从节点112操作：
安装数据库：
[root@node2 ~]# yum -y install mariadb-server

修改配置文件：
[root@node2 ~]# cat /etc/my.cnf
[mysqld]
server-id=2
read-only

启动mysql数据库
[root@node2 ~]# systemctl start mariadb
[root@node2 ~]# mysql
MariaDB [(none)]> CHANGE MASTER TO
    -> MASTER_HOST='192.168.0.111',
    -> MASTER_USER='repluser',
    -> MASTER_PASSWORD='replpass',
    -> MASTER_PORT=3306,
    -> MASTER_LOG_FILE='mariadb-bin.000001',
    -> MASTER_LOG_POS=245;
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G
MariaDB [(none)]> select user,host,password from mysql.user;

repluser | 192.168.0.% | *D98280F03D0F78162EBDBB9C883FC01395DEA2BF


从节点113操作：
[root@node3 ~]# yum -y install mariadb-server

修改配置文件
[root@node3 ~]# cat /etc/my.cnf
[mysqld]
server-id=3
read-only

启动mysql数据库
[root@node3 ~]# systemctl start mariadb
[root@node3 ~]# mysql
MariaDB [(none)]> CHANGE MASTER TO
    -> MASTER_HOST='192.168.0.111',
    -> MASTER_USER='repluser',
    -> MASTER_PASSWORD='replpass',
    -> MASTER_PORT=3306,
    -> MASTER_LOG_FILE='mariadb-bin.000001',
    -> MASTER_LOG_POS=245;
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G
MariaDB [(none)]> select user,host,password from mysql.user;

主节点111操作：
MariaDB [(none)]> create database db1;
从节点112和113 操作
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| mysql              |
| performance_schema |
| test               |
+--------------------+

主节点111操作：
安装semisync 插件
MariaDB [(none)]> install plugin rpl_semi_sync_master SONAME 'semisync_master.so';
查看安装的插件：
MariaDB [(none)]> show plugins;
查看semi变量
MariaDB [(none)]> show global variables like '%semi%';
+------------------------------------+-------+
| Variable_name                      | Value |
+------------------------------------+-------+
| rpl_semi_sync_master_enabled       | OFF   |
| rpl_semi_sync_master_timeout       | 10000 |
| rpl_semi_sync_master_trace_level   | 32    |
| rpl_semi_sync_master_wait_no_slave | ON    |
+------------------------------------+-------+
4 rows in set (0.00 sec)

启动semi
MariaDB [(none)]> set global rpl_semi_sync_master_enabled=ON;

查看semi状态变量：
MariaDB [(none)]> show global status like '%semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 0     |
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 0     |
| Rpl_semi_sync_master_tx_wait_time          | 0     |
| Rpl_semi_sync_master_tx_waits              | 0     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 0     |
+--------------------------------------------+-------+
14 rows in set (0.00 sec)

从节点112操作：
安装客户端semisync
MariaDB [(none)]> install plugin rpl_semi_sync_slave SONAME 'semisync_slave.so';

查看变量
MariaDB [(none)]>  show global variables like '%semi%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | OFF   |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+
2 rows in set (0.00 sec)

启动semisync
MariaDB [(none)]> set global rpl_semi_sync_slave_enabled=ON;

重启线程：
MariaDB [(none)]> stop slave;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> show global status like '%semi%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Rpl_semi_sync_slave_status | ON    |
+----------------------------+-------+
1 row in set (0.00 sec)


从节点113操作：
安装客户端semisync
MariaDB [(none)]> install plugin rpl_semi_sync_slave SONAME 'semisync_slave.so';

查看变量
MariaDB [(none)]>  show global variables like '%semi%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | OFF   |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+
2 rows in set (0.00 sec)

启动semisync
MariaDB [(none)]> set global rpl_semi_sync_slave_enabled=ON;

重启线程：
MariaDB [(none)]> stop slave;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> show global status like '%semi%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Rpl_semi_sync_slave_status | ON    |
+----------------------------+-------+
1 row in set (0.00 sec)

测试主节点创建db2数据库
MariaDB [(none)]> create database db2;

从节点验证
MariaDB [(none)]> show databases;
 db2    
 
把从节点112mysql服务停止：
[root@node2 ~]# systemctl stop mariadb
测试主节点创建db3数据库：
MariaDB [(none)]> create database db3;
从节点113验证数据库是否存在
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| db2                |
| db3                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
7 rows in set (0.00 sec)

在把从节点113mysql服务停止
[root@node3 ~]# systemctl stop mariadb
测试主节点创建db4数据库:
MariaDB [(none)]> create database db4;
就不能创建成功....超时10秒断开

从节点mysql服务重新启动：
[root@node2 ~]# systemctl start mariadb
[root@node3 ~]# systemctl start mariadb
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| db2                |
| db3                |
| db4                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
8 rows in set (0.00 sec)


总结： 半同步复制
三台主机
先配置主从 
安装semisync 插件 并且启动

```

## 1.2.1 复制过滤器

```
环境准备2台主机 做主从
111，112 
111主节点安装mysql
[root@node1 ~]# yum -y install mariadb-server

修改配置文件：
[root@node1 ~]# cat /etc/my.cnf
[mysqld]
server-id=1
log-bin
binlog-ignore-db=hellodb

启动mysql
[root@node1 ~]# systemctl start mariadb
[root@node1 ~]# mysql
MariaDB [(none)]> grant replication slave on *.* to repluser@'192.168.0.%' identified by 'replpass';
MariaDB [(none)]> flush privileges;

从节点112 安装mariadb-server
[root@node2 ~]# yum -y install mariadb-server

修改配置文件
[root@node2 ~]# cat /etc/my.cnf
[mysqld]
server-id=2
read-only

启动mysql服务
[mysqld]
server-id=2
read-only
[root@node2 ~]# systemctl start mariadb

关联授权主机和用户
[root@node2 ~]# mysql
MariaDB [(none)]> CHANGE MASTER TO
    -> MASTER_HOST='192.168.0.111',
    -> MASTER_USER='repluser',
    -> MASTER_PASSWORD='replpass',
    -> MASTER_PORT=3306,
    -> MASTER_LOG_FILE='mariadb-bin.000001',
    -> MASTER_LOG_POS=245;
    
启动复制线程：
MariaDB [(none)]> start slave;
[root@node2 ~]# mysql -e 'show slave status\G' |grep Yes
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

测试主库先创建一个数据库db1；
MariaDB [(none)]> create database db1;

从库验证是否创建成功：
[root@node2 ~]# mysql -e 'show databases'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| mysql              |
| performance_schema |
| test               |
+--------------------+

测试导入hellodb数据库sql文件：
MariaDB [(none)]> \! ls
hellodb_innodb.sql
MariaDB [(none)]> source /root/hellodb_innodb.sql

从库验证，配置是否生效：
[root@node2 ~]# mysql -e 'show databases'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| mysql              |
| performance_schema |
| test               |
+--------------------+

```

### 1.2.2 复制过滤器 第二种方法

```
环境准备2台主机 做主从
111，112 
111主节点安装mysql
[root@node1 ~]# yum -y install mariadb-server

修改配置文件：
[root@node1 ~]# cat /etc/my.cnf
[mysqld]
server-id=1
log-bin

启动mysql
[root@node1 ~]# systemctl start mariadb
[root@node1 ~]# mysql
MariaDB [(none)]> grant replication slave on *.* to repluser@'192.168.0.%' identified by 'replpass';
MariaDB [(none)]> flush privileges;

从节点112 安装mariadb-server
[root@node2 ~]# yum -y install mariadb-server

修改配置文件
[root@node2 ~]# cat /etc/my.cnf
[mysqld]
server-id=2
read-only
replicate-do-db=hellodb

启动mysql服务
[mysqld]
server-id=2
read-only
[root@node2 ~]# systemctl start mariadb

关联授权主机和用户
[root@node2 ~]# mysql
MariaDB [(none)]> CHANGE MASTER TO
    -> MASTER_HOST='192.168.0.111',
    -> MASTER_USER='repluser',
    -> MASTER_PASSWORD='replpass',
    -> MASTER_PORT=3306,
    -> MASTER_LOG_FILE='mariadb-bin.000001',
    -> MASTER_LOG_POS=245;
    
启动复制线程：
MariaDB [(none)]> start slave;
[root@node2 ~]# mysql -e 'show slave status\G' |grep Yes
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            
测试主库导入hellodb文件：
[root@node1 ~]# mysql  < hellodb_innodb.sql 
[root@node1 ~]# mysql -e 'show databases’

+--------------------+
| Database           |
+--------------------+
| information_schema |
| hellodb            |
| mysql              |
| performance_schema |
| test               |
+--------------------+

验证从库是否存在hellodb
[root@node2 ~]# mysql -e 'show databases'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hellodb            |
| mysql              |
| performance_schema |
| test               |
+--------------------+

主库创建一个db1：
[root@node1 ~]# mysql -e 'create database db1'
[root@node1 ~]# mysql -e 'show databases'
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

从库验证：
[root@node2 ~]# mysql -e 'show databases'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hellodb            |
| mysql              |
| performance_schema |
| test               |
+--------------------+

如果不进入hellodb数据库插入数据，过滤器不会生效；
主库操作：
MariaDB [(none)]> insert hellodb.teachers (name,age) values('wang',30);
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> select * from hellodb.teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   5 | wang          |  30 | NULL   |
+-----+---------------+-----+--------+
5 rows in set (0.00 sec)

从库查询：
MariaDB [hellodb]> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
+-----+---------------+-----+--------+
4 rows in set (0.00 sec)

主库进入hellod插入数据：
MariaDB [hellodb]> insert teachers (name,age) values ('mage',30);
Query OK, 1 row affected (0.00 sec)

MariaDB [hellodb]> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   5 | wang          |  30 | NULL   |
|   6 | mage          |  30 | NULL   |
+-----+---------------+-----+--------+
6 rows in set (0.00 sec)

从库验证：
MariaDB [hellodb]> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   6 | mage          |  30 | NULL   |
+-----+---------------+-----+--------+
5 rows in set (0.00 sec)

总结：在主库操作时，要进入相应的数据库，过滤器的功能才会起作用；
```

## 1.2.3 mysql复制加密

```
主节点111操作：
生成私钥文件：
[root@node1 ~]# mkdir /etc/my.cnf.d/ssl
[root@node1 ~]# cd /etc/my.cnf.d/ssl
[root@node1 ssl]# (umask 066;openssl genrsa -out cakey.pem 2048)

生成自签请求：
[root@node1 ssl]# openssl req -new -x509 -key cakey.pem -out cacert.pem -days 3650

生成master私钥和签署请求：
[root@node1 ssl]# openssl req -newkey rsa:1024 -days 365 -nodes -keyout master.key > master.csr
[root@node1 ssl]# ls
cacert.pem  cakey.pem  master.csr  master.key

签署master请求：
[root@node1 ssl]# openssl x509 -req -in master.csr -CA cacert.pem -CAkey cakey.pem -set_serial 01 >master.crt

生成slave私钥和签署请求：
[root@node1 ssl]# openssl req -newkey rsa:1024 -days 365 -nodes -keyout slave.key > slave.csr

签署slave请求：
[root@node1 ssl]# openssl x509 -req -in slave.csr -CA cacert.pem -CAkey cakey.pem -set_serial 02 >slave.crt

[root@node1 ssl]# ll
total 32
-rw-r--r--. 1 root root 1424 Nov 28 07:02 cacert.pem
-rw-------. 1 root root 1675 Nov 28 07:00 cakey.pem
-rw-r--r--. 1 root root 1123 Nov 28 07:10 master.crt
-rw-r--r--. 1 root root  708 Nov 28 07:05 master.csr
-rw-r--r--. 1 root root  916 Nov 28 07:05 master.key
-rw-r--r--. 1 root root 1119 Nov 28 07:14 slave.crt
-rw-r--r--. 1 root root  704 Nov 28 07:13 slave.csr
-rw-r--r--. 1 root root  916 Nov 28 07:13 slave.key

MariaDB [(none)]> status;
SSL:			Not in use
MariaDB [(none)]> show variables like '%ssl%';
| Variable_name | Value    |
+---------------+----------+
| have_openssl  | DISABLED |
| have_ssl      | DISABLED |
| ssl_ca        |          |
| ssl_capath    |          |
| ssl_cert      |          |
| ssl_cipher    |          |
| ssl_key       |          |


[root@node1 ssl]# cat /etc/my.cnf
[mysqld]
server-id=1
log-bin
ssl-ca=/etc/my.cnf.d/ssl/cacert.pem
ssl-cert=/etc/my.cnf.d/ssl/master.crt
ssl-key=/etc/my.cnf.d/ssl/master.key

[root@node1 ssl]# systemctl restart mariadb

MariaDB [(none)]> show variables like '%ssl%';
+---------------+------------------------------+
| Variable_name | Value                        |
+---------------+------------------------------+
| have_openssl  | YES                          |
| have_ssl      | YES                          |
| ssl_ca        | /etc/my.cnf.d/ssl/cacert.pem |
| ssl_capath    |                              |
| ssl_cert      | /etc/my.cnf.d/ssl/master.crt |
| ssl_cipher    |                              |
| ssl_key       | /etc/my.cnf.d/ssl/master.key |
+---------------+------------------------------+

使用ssl登录
[root@node1 ssl]# mysql --ssl-ca=cacert.pem --ssl-cert=master.crt --ssl-key=master.key
MariaDB [(none)]> status;
SSL:			Cipher in use is DHE-RSA-AES256-GCM-SHA384

复制ssl 目录到从库
[root@node1 ssl]# scp -r /etc/my.cnf.d/ssl/ 192.168.0.112:/etc/my.cnf.d/

从库112操作：
[root@node2 ssl]# mysql --ssl-ca=cacert.pem --ssl-cert=slave.crt --ssl-key=slave.key -h192.168.0.111 -urepluser -preplpass
MariaDB [(none)]> status;
SSL:			Cipher in use is DHE-RSA-AES256-GCM-SHA384


授权一个强制加密远程登录的账号
主库111操作：
MariaDB [(none)]> grant replication slave on *.* to repluser2@'192.168.0.%' identified by 'replpass' require ssl;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;

从库112配置ssl：
[root@node2 ~]# cat /etc/my.cnf
[mysqld]
server-id=2
read-only
#replicate-do-db=hellodb
ssl-ca=/etc/my.cnf.d/ssl/cacert.pem
ssl-cert=/etc/my.cnf.d/ssl/slave.crt
ssl-key=/etc/my.cnf.d/ssl/slave.key

root@node2 ssl]# systemctl restart mariadb

删除原来的复制账号
MariaDB [(none)]> stop slave;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> reset slave all;
Query OK, 0 rows affected (0.00 sec)

关联复制账号和主机：
MariaDB [(none)]> CHANGE MASTER TO
    -> MASTER_HOST='192.168.0.111',
    -> MASTER_USER='repluser2',
    -> MASTER_PASSWORD='replpass',
    -> MASTER_PORT=3306,
    -> MASTER_LOG_FILE='mariadb-bin.000002',   查看主库的二进制文件postion位置
    -> MASTER_LOG_POS=491,
    -> MASTER_SSL=1;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G

测试主库删除hellodb
MariaDB [(none)]> drop database hellodb;

验证从库是否存在
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db2                |
| mysql              |
| performance_schema |
| test               |
+--------------------+






```

## 1.2.4  二进制安装mysql启动GTID复制

```
下载mysql-5.7.26-el7-x86_64.tar.gz； 5.6之后的才支持GTID
解压mysql安装包
[root@node2 ~]# tar xf mysql-5.7.26-el7-x86_64.tar.gz -C /usr/local/

创建mysql用户
[root@node2 ~]# useradd -r -s /bin/false mysql

创建软链接
[root@node2 local]# cd /usr/local
[root@node2 local]# ln -s mysql-5.7.26-el7-x86_64/ mysql

导入环境变量
[root@node2 local]# echo 'PATH=/usr/local/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
[root@node2 local]# . /etc/profile.d/mysql.sh

创建数据目录
[root@node2 local]# mkdir -p /data/mysql
[root@node2 local]# chown mysql.mysql /data/mysql

初始化脚本
[root@node2 local]# mysqld --initialize --user=mysql --datadir=/data/mysql
 password is generated for root@localhost: *1uGf-y7>tLA

备份配置文件并且修改
[root@node2 local]# cp /etc/my.cnf{,.bak}
[root@node2 local]# vim /etc/my.cnf
[root@node2 local]# cat /etc/my.cnf
[mysqld]
datadir=/data/mysql
socket=/data/mysql/mysql.sock
log-error=/data/mysql/mysql.log
pid-file=/data/mysql/mysql.pid

[client]
socket=/data/mysql/mysql.sock

复制启动脚本
[root@node2 local]# cp mysql/support-files/mysql.server /etc/init.d/mysqld

加入自启动列表
[root@node2 local]# chkconfig --add mysqld
[root@node2 local]# chkconfig --list

启动服务
[root@node2 local]# service mysqld start 

连接数据库
[root@node2 local]# mysql -p"*1uGf-y7>tLA"

修改密码
[root@node2 local]# mysqladmin -uroot -p"*1uGf-y7>tLA" password 'shaoxh'

测试导入hellodb数据
[root@node2 local]# mysql -pshaoxh < hellodb_innodb.sql 
[root@node2 local]# mysql -pshaoxh -e "show databases"
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hellodb            |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

查看数据密码
mysql> select user,host,authentication_string from user;
+---------------+-----------+-------------------------------------------+
| user          | host      | authentication_string                     |
+---------------+-----------+-------------------------------------------+
| root          | localhost | *83E4ED024D412E460916C26014FF8C4ECD68B047 |
| mysql.session | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys     | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
+---------------+-----------+-------------------------------------------+
3 rows in set (0.00 sec)


从节点113 也需要安装
password is generated for root@localhost: sIEwhqQir2_U

[root@node3 local]# mysql -p"sIEwhqQir2_U"
[root@node3 local]# mysqladmin -uroot -p"sIEwhqQir2_U" password "shaoxh"



```

### 1.2.5 配置GTID主从复制

```
主库的配置文件
[root@node2 local]# cat /etc/my.cnf
[mysqld]
server-id=1
log-bin
gtid-mode=ON
enforce-gtid-consistency

授权账号
mysql> grant replication slave on *.* to repluser@'192.168.0.%' identified by 'replpass';
mysql> flush privileges;

重启mysql
[root@node2 local]# service mysqld restart

从库配配置文件
[root@node3 local]# cat /etc/my.cnf
[mysqld]
server-id=2
read-only
gtid-mode=on
enforce-gtid-consistency

重启mysql
[root@node3 local]# service mysqld restart

关联账号和主机
mysql> CHANGE MASTER TO
    -> MASTER_HOST='192.168.0.112',
    -> MASTER_USER='repluser',
    -> MASTER_PASSWORD='replpass',
    -> MASTER_PORT=3306,
    -> MASTER_AUTO_POSITION=1;
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G

测试主库创建db1
mysql> create database db1;

验证从库是否存在db1
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| mysql              |
| performance_schema |
| sys                |
+--------------------+




```

## 1.2.6 基于proxySQL实现mysql的读写分离

```
准备环境：3台主机
111主 112从 113proxy

主节点111安装mysql
[root@node1 ~]# yum -y install mariadb-server

修改配置文件
[root@node1 ~]# cat /etc/my.cnf
server-id=1
log-bin

启动数据库
[root@node1 ~]# systemctl start mariadb

授权远程连接用户
[root@node1 ~]# mysql -e "grant replication slave on *.* to repluser@'192.168.0.%' identified by 'replpass'"

112从节点安装mysql
[root@node2 ~]# yum -y install mariadb-server

修改配置文件
[root@node2 ~]# cat /etc/my.cnf
server-id=2
read-only

启动数据库
[root@node2 ~]# systemctl start mariadb

关联用户和主机
[root@node2 ~]# mysql -e "CHANGE MASTER TO MASTER_HOST='192.168.0.111',MASTER_USER='repluser',
> MASTER_PASSWORD='replpass',MASTER_PORT=3306,
> MASTER_LOG_FILE='mariadb-bin.000001',MASTER_LOG_POS=245;”
[root@node2 ~]# mysql -e "start slave"
[root@node2 ~]# mysql -e "show slave status\G"

主节点导入hellodb文件
[root@node1 ~]# mysql < hellodb_innodb.sql 

从节点验证数据库
[root@node2 ~]# mysql -e "show databases" |grep hellodb
hellodb

113节点安装proxysql
配置yum源
[root@node3 ~]# 
cat <<EOF | tee /etc/yum.repos.d/proxysql.repo
[proxysql_repo]
name= ProxySQL YUM repository
baseurl=https://repo.proxysql.com/ProxySQL/proxysql-2.0.x/centos/\$releasever
gpgcheck=1
gpgkey=https://repo.proxysql.com/ProxySQL/repo_pub_key
EOF

安装proxysql 
[root@node3 ~]#  yum -y install proxysql

修改端口成为3306
[root@node3 ~]# cat /etc/proxysql.cnf
mysql_variables=
         ....
         3306
         ....
启动proxysql
[root@node3 ~]# systemctl start proxysql

连接proxy管理接口
[root@node3 ~]# yum -y install mariadb
[root@node3 ~]# mysql -uadmin -padmin -P6032 -h127.0.0.1
MySQL [(none)]> show databases;
MySQL [(none)]> show tables;
MySQL [(none)]> show tables from monitor;

查看表的结构
MySQL [(none)]> select * from sqlite_master where name='mysql_servers'\G
表中的数据
MySQL [(none)]> select * from mysql_servers;
Empty set (0.00 sec)

插入数据
MySQL [(none)]> insert into mysql_servers(hostgroup_id,hostname,port) values(10,'192.168.0.111',3306);


MySQL [(none)]> insert into mysql_servers(hostgroup_id,hostname,port) values(10,'192.168.0.112',3306);

加载内存生效；保存磁盘
MySQL [(none)]> load mysql servers to runtime;
MySQL [(none)]> save mysql servers to disk;

主节点创建监控的账号
[root@node1 ~]# mysql -e "grant replication client on *.* to monitor@'192.168.0.%' identified by 'shaoxh'"
[root@node1 ~]# mysql -e "flush privileges"

proxysql上配置监控
MySQL [(none)]> set mysql-monitor_username='monitor';
MySQL [(none)]> set mysql-monitor_password='shaoxh';

加上在到runtime ，并保存到disk
MySQL [(none)]> load mysql variables to runtime;
MySQL [(none)]> save mysql variables to disk;

查看监控连接是否正常
MySQL [(none)]> select * from mysql_server_connect_log;

查看监控心跳信息
MySQL [(none)]> select * from mysql_server_ping_log;

查看read_only和relication_lag的监控日志
MySQL [(none)]> select * from mysql_server_read_only_log;
Empty set (0.00 sec)

MySQL [(none)]> select * from mysql_server_replication_lag_log;
Empty set (0.00 sec)

MySQL [(none)]> 

设置分组信息
MySQl [(none)]> insert into mysql_replication_hostgroups values (10,20,'read_only','test');
MySQL [(none)]> load mysql servers to runtime;
MySQL [(none)]> save mysql servers to disk;

Monitor 模块监控后端的read_only值，自动移动到读写组

MySQL [(none)]> select hostgroup_id,hostname,port,status,weight from mysql_servers;
+--------------+---------------+------+--------+--------+
| hostgroup_id | hostname      | port | status | weight |
+--------------+---------------+------+--------+--------+
| 10           | 192.168.0.111 | 3306 | ONLINE | 1      |
| 20           | 192.168.0.112 | 3306 | ONLINE | 1      |
+--------------+---------------+------+--------+--------+
2 rows in set (0.00 sec)

配置发送SQL语句的用户
在master节点上创建访问用户
[root@node1 ~]# mysql -e "grant all on *.* to sqluser@'192.168.0.%' identified by 'shaoxh'"

在proxy 配置，将用户sqluser添加到mysql_users表中，default_hostgoup默认组设置为写组10，当读写分离的路由规则不符合时，会访问默认组的数据库
MySQL [(none)]> insert into mysql_users(username,password,default_hostgroup) values('sqluser','shaoxh',10);


MySQL [(none)]> load mysql users to runtime;


MySQL [(none)]> save mysql users to disk;


使用sqluser用户测试是否能路由到默认的10写组实现，读、写数据
[root@node3 ~]# mysql -usqluser -pshaoxh -P3306 -h127.0.0.1 -e 'select @@server_id'
[root@node3 ~]# mysql -usqluser -pshaoxh -P3306 -h127.0.0.1 -e 'create database db1'

在proxysql上配置路由规则
MySQL [(none)]> insert into mysql_query_rules
    -> (rule_id,active,match_digest,destination_hostgroup,apply) values
    -> (1,1,'^select .* for update$',10,1),(2,1,'^select',20,1);

MySQL [(none)]> load mysql query rules to runtime;
MySQL [(none)]> save mysql query rules to disk;

MySQL [(none)]> select rule_id,active,match_digest,destination_hostgroup,apply from mysql_query_rules;
+---------+--------+------------------------+-----------------------+-------+
| rule_id | active | match_digest           | destination_hostgroup | apply |
+---------+--------+------------------------+-----------------------+-------+
| 1       | 1      | ^select .* for update$ | 10                    | 1     |
| 2       | 1      | ^select                | 20                    | 1     |
+---------+--------+------------------------+-----------------------+-------+
2 rows in set (0.00 sec)

[root@node3 ~]# mysql -usqluser -pshaoxh -P3306 -h127.0.0.1 -e 'select @@server_id'
+-------------+
| @@server_id |
+-------------+
|           2 |
+-------------+

[root@node3 ~]# mysql -usqluser -pmagedu -P3306 -h127.0.0.1 -e 'begin;select @@server_id;commit'
+-------------+
| @@server_id |
+-------------+
|           1 |
+-------------+

[root@node3 ~]# mysql -usqluser -pmagedu -P3306 -h127.0.0.1 -e 'create table t1 (id int);' db1

主节点查看是否创建成功
[root@node1 ~]# mysql -e "use db1;show tables"
+---------------+
| Tables_in_db1 |
+---------------+
| t1            |
+---------------+

路由的信息：查看stats库中stats_mysql_query_digest表
MySQL [db1]> select hostgroup hg,sum_time,count_star, digest_text from stats_mysql_query_digest order by sum_time desc;
+----+----------+------------+----------------------------------+
| hg | sum_time | count_star | digest_text                      |
+----+----------+------------+----------------------------------+
| 10 | 4735     | 1          | create table t1 (id int)         |
| 10 | 1569     | 2          | select @@server_id               |
| 20 | 1459     | 1          | select @@server_id               |
| 10 | 1064     | 1          | create database db1              |
| 10 | 497      | 1          | commit                           |
| 10 | 337      | 1          | begin                            |
| 10 | 0        | 1          | select @@version_comment limit ? |
| 10 | 0        | 4          | select @@version_comment limit ? |
+----+----------+------------+----------------------------------+





```

## 1.2.7 MHA

```
环境准备
四台主机，node{1，2，3，4}
mha管理节点node1，主从234 主为2，从为，2，3

生成密钥对，使所有主机都能互相通信
ssh-keygen -t rsa  
cat .ssh/id_rsa.pub > .ssh/authorized_keys
 chmod go= .ssh/authorized_keys 
把私钥复制到其他的节点上去
scp -p .ssh/* 192.168.111.128:/root/.ssh
验证

ssh 192.168.111.128 'ifconfig'

node2节点安装mariadb-server
[root@node2 ~]# yum -y install mariadb-server

修改配置文件
[root@node2 ~]# cat /etc/my.cnf
[mysqld]
server-id=112
log-bin
skip_name_resolve

启动mariadb
[root@node2 ~]# systemctl start mariadb

node3节点安装mariadb-server
[root@node3 ~]# yum -y install mariadb-server

修改配置文件
[root@node3 ~]# cat /etc/my.cnf
[mysqld]
server-id=113
log-bin
read-only
skip-name-resolve
relay-log-purge=0


启动mariadb
[root@node3 ~]# systemctl start mariadb


node4节点安装mariadb-server
[root@node4 ~]# yum -y install mariadb-server

修改配置文件
[root@node4 ~]# cat /etc/my.cnf
[mysqld]
server-id=114
log-bin
read-only
skip-name-resolve
relay-log-purge=0

启动数据库
[root@node4 ~]# systemctl start mariadb

主库授权远程用户
[root@node2 ~]# mysql -e "grant replication slave on *.* to repluser@'192.168.0.%' identified by 'replpass'"
[root@node2 ~]# mysql -e "grant all on *.* to mhauser@'192.168.0.%' identified by 'shaoxh'"
[root@node2 ~]# mysql -e "flush privileges"

从库113关联
[root@node3 ~]# mysql -e "CHANGE MASTER TO MASTER_HOST='192.168.0.112',
MASTER_USER='repluser',
MASTER_PASSWORD='replpass',
MASTER_PORT=3306,
MASTER_LOG_FILE='mariadb-bin.000001',
MASTER_LOG_POS=245"

[root@node3 ~]# mysql -e "start slave"
[root@node3 ~]# mysql -e "show slave status\G"|grep Yes
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

从库114关联：
[root@node4 ~]# mysql -e "CHANGE MASTER TO MASTER_HOST='192.168.0.112',
MASTER_USER='repluser',
MASTER_PASSWORD='replpass',
MASTER_PORT=3306,
MASTER_LOG_FILE='mariadb-bin.000001',
MASTER_LOG_POS=245"

[root@node4 ~]# mysql -e "start slave"
[root@node4 ~]# mysql -e "show slave status\G"|grep Yes
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

实现MHA 
在管理节点上安装两个包
mha4mysql-manager
mha4mysql-node

在被管理节点安装
mha4mysql-node

node1管理节点上传两个rpm
[root@node1 ~]# ll mha*
-rw-r--r--. 1 root root 87119 Nov 30 01:28 mha4mysql-manager-0.56-0.el6.noarch.rpm
-rw-r--r--. 1 root root 36326 Nov 30 01:28 mha4mysql-node-0.56-0.el6.noarch.rpm

安装mha，有依赖关系，需要epel源
[root@node1 ~]# yum -y install epel*
[root@node1 ~]# yum -y install mha*

被管理节点都都要安装mha-node
[root@node3 ~]# ll mha*
-rw-r--r--. 1 root root 36326 Nov 30 01:28 mha4mysql-node-0.56-0.el6.noarch.rpm
[root@node3 ~]# yum -y install epel*
[root@node3 ~]# yum -y install mha*

管理节点创建配置文件
[root@node1 ~]# mkdir /etc/mha
[root@node1 ~]# cat /etc/mha/app1.cnf
[server default]
user=mhauser
password=shaoxh
manager_workdir=/data/mha/app1/
manager_log=/data/mha/app1/manager.log
remote_workdir=/data/mha/app1/
ssh_user=root
repl_user=repluser
repl_password=replpass
ping_interval=1

[server1]
hostname=192.168.0.112
candidate_master=1

[server2]
hostname=192.168.0.113
candidate_master=1

[server3]
hostname=192.168.0.114

启动配置文件
[root@node1 ~]# masterha_check_ssh --conf=/etc/mha/app1.cnf
[root@node1 ~]# masterha_check_repl --conf=/etc/mha/app1.cnf
[root@node1 ~]# masterha_manager --conf=/etc/mha/app1.cnf

测试主库执行导入数据中停止数据库
[root@node2 ~]# mysql test < testlog.sql
MariaDB [test]> call pro_testlog;

管理节点就会自动断开连接，也有错误日志
[root@node1 ~]# tail -f /data/mha/app1/manager.log

Master 192.168.0.112(192.168.0.112:3306) is down!
Master failover to 192.168.0.113(192.168.0.113:3306) completed successfully.

在新的主库创建db1
MariaDB [(none)]> create database db1;
MariaDB [(none)]> show variables like 'read_only';

从库114 验证
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| mysql              |
| performance_schema |
| test               |
+--------------------+

最后把113新主库配置文件的read-only注释
[root@node3 ~] vim /etc/my.cnf
#read-only




```

## 1.2.8 galeacluster 集群

```
环境准备：
三台主机 node1 node2 node3
配置maridb源
[root@node1 ~]# cat > /etc/yum.repos.d/MariDB.repo <<EOF
[MariDB]
name=MariDB-5.5.68
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mariadb/mariadb-5.5.68/yum/centos7-amd64/
gpgcheck=0
EOF
```

