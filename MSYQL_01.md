# MSYQL 01

## 1.1.1 修改密码

```
/usr/bin/mysqladmin -u root password 'new-password'
/usr/bin/mysql_secure_installation

```

### 1.1.2  查看数据库状态

```
MariaDB [(none)]> status

```

### 1.1.3 切换数据库

```
MariaDB [(none)]> use mysql

```

### 1.1.4 查看表结构

```
MariaDB [mysql]> desc user;

```

### 1.1.5 查看字段

```
MariaDB [mysql]> select user,host,password from user;
+------+-----------+----------+
| user | host      | password |
+------+-----------+----------+                
| root | localhost |          |               
| root | node2     |          |
| root | 127.0.0.1 |          |
| root | ::1       |          |
|      | localhost |          |
|      | node2     |          |
+------+-----------+----------+
6 rows in set (0.00 sec)

```

```
user 是本机数据库的用户
host 客户端的主机名称
空是 null
```

### 1.1.6 匿名用户

```
[root@node2 ~]# mysql -u xxx
MariaDB [(none)]> status
--------------
mysql  Ver 15.1 Distrib 10.2.35-MariaDB, for Linux (x86_64) using readline 5.1

Connection id:		13
Current database:	
Current user:		xxx@localhost
SSL:			Not in use
Current pager:		stdout


MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| test               |
+--------------------+
2 rows in set (0.00 sec)

```

### 1.1.7 安全加固

```
[root@node2 ~]# /usr/bin/mysql_secure_installation 

```

## 2.1.1 mysqladmin命令

```
[root@node1 ~]# mysqladmin --help
mysqladmin  Ver 9.0 Distrib 5.5.65-MariaDB, for Linux on x86_64
Copyright (c) 2000, 2018, Oracle, MariaDB C
[root@node1 ~]# mysqladmin shutdown
```

### 2.1.2 ping

```
[root@node1 ~]# systemctl start mariadb
[root@node1 ~]# mysqladmin ping
mysqld is alive

```

### 2.1.3 passoword

```
[root@node1 ~]# mysqladmin password shaoxh
[root@node1 ~]# mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
[root@node1 ~]# mysql -pshaoxh
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 7
Server version: 5.5.65-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

```

### 2.1.4 查看user表的字段

```
MariaDB [(none)]> select user,host,password from mysql.user;
+------+-----------+-------------------------------------------+
| user | host      | password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | *83E4ED024D412E460916C26014FF8C4ECD68B047 |
| root | node1     |                                           |
| root | 127.0.0.1 |                                           |
| root | ::1       |                                           |
|      | localhost |                                           |
|      | node1     |                                           |
+------+-----------+-------------------------------------------+
6 rows in set (0.00 sec)

```

### 2.1.4 修改密码

```
[root@node1 ~]# mysqladmin -uroot -pshaoxh password magedu
[root@node1 ~]# mysql -pshaoxh
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
[root@node1 ~]# mysql -pmagedu
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 5.5.65-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

```

## 2.1.5 删除msql目录下文件 （测试)

```
[root@node1 ~]# ls /var/lib/mysql/
aria_log.00000001  aria_log_control  ibdata1  ib_logfile0  ib_logfile1  mysql  mysql.sock  performance_schema  test
[root@node1 ~]# rm -rf /var/lib/mysql/*
[root@node1 ~]# systemctl restart mariadb
[root@node1 ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.65-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

```

### 2.1.6 创建数据库

```
[root@node1 ~]# mysqladmin create db2
[root@node1 ~]# mysql -e "show databases"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| db2                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
[root@node1 ~]# 

```

### 2.1.7 文件导入方式执行

```
[root@node1 ~]# cat > test.sql
use mysql
show databases;
select user,host,password from user;
^C
[root@node1 ~]# mysql < test.sql 
Database
information_schema
db1
db2
mysql
performance_schema
test
user	host	password
root	localhost	
root	node1	
root	127.0.0.1	
root	::1	
	localhost	
	node1	
[root@node1 ~]# 


[root@node1 ~]# cat test.sql |mysql
Database
information_schema
db1
db2
mysql
performance_schema
test
user	host	password
root	localhost	
root	node1	
root	127.0.0.1	
root	::1	
	localhost	
	node1	
[root@node1 ~]# 

```

### 2.1.8 查看父子进程

```
                                  
[root@node1 ~]# ps auxf
[root@node1 ~]# pstree -p
脚本文件
[root@node1 ~]# rpm -q --scripts mariadb-server
多实例
[root@node1 ~]# rpm -ql mariadb-server|grep multi
/usr/bin/mysqld_multi
 

```

### 2.1.9 system 命令

```
MariaDB [(none)]> system hostname
node1
MariaDB [(none)]> system date
Thu Nov  5 08:26:50 EST 2020
MariaDB [(none)]> 
执行shell中的命令
```

2.2.1 source 

```
MariaDB [(none)]> source test.sql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| db2                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.00 sec)

```

## 2.2.1 修个mysql登录提示符

```
[root@node5 ~]#  export MYSQL_PS1="(\u@\h) [\d]> "
[root@node5 ~]# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

(root@localhost) [(none)]> 

```

```

[root@node1 ~]# grep -C 2 prompt  /etc/my.cnf.d/mysql-clients.cnf

[mysql]
prompt="\\r:\\m:\\s>"
[mysql_upgrade]


[root@node1 ~]# mysql --help
[root@node1 ~]# mysql --print-defaults
mysql would have been started with the following arguments:
--prompt=\r:\m:\s:\N [\d]> 

```

### 2.2.2 直接进入mysql

```
[root@node2 ~]# mysql -D mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
[root@node2 ~]# mysql -D mysql -pshaoxh
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 33
Server version: 10.2.35-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [mysql]> 

MariaDB [(none)]> select version();       
+-----------------+
| version()       |
+-----------------+
| 10.2.35-MariaDB |
+-----------------+
1 row in set (0.00 sec)


```

### 2.2.3 删除库

```
[root@node1 ~]# mysqladmin drop db1;
Dropping the database is potentially a very bad thing to do.
Any data stored in the database will be destroyed.

Do you really want to drop the 'db1' database [y/N] y
Database "db1" dropped
[root@node1 ~]# mysql -e "show databases
> "
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db2                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
[root@node1 ~]# 

```

### 2.2.4 禁止ip登录，用与维护

```
[root@node1 mysql]# grep -C 2 skip /etc/my.cnf
[mysqld]
skip-networking=1
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
[root@node1 mysql]# 

```

## yum安装只适合测试，二进制和源码安装用于生产环境

3.1.1 二进制安装mysql

```
[root@node1 data]# mkdir /data/mysql -p
[root@node1 data]# useradd -r -s /sbin/nologin -d /data/mysql/ mysql
[root@node1 data]# chown mysql:mysql /data/mysql/

```

### 3.1.2 解压缩mariadb

```
[root@node1 ~]# tar xf mariadb-10.2.25-linux-x86_64.tar.gz -C /usr/local/
[root@node1 ~]# ln -sv /usr/local/mariadb-10.2.25-linux-x86_64/ /usr/local/mysql
[root@node1 ~]# cd /usr/local
[root@node1 local]# chown -R root.root mysql/
[root@node1 mysql]# scripts/mysql_install_db --datadir=/data/mysql --user=mysql
Installing MariaDB/MySQL system tables in '/data/mysql' ...
OK

[root@node1 mysql]# ll /data/mysql/
total 110620
-rw-rw----. 1 mysql mysql    16384 Nov  5 20:17 aria_log.00000001
-rw-rw----. 1 mysql mysql       52 Nov  5 20:17 aria_log_control
-rw-rw----. 1 mysql mysql      938 Nov  5 20:17 ib_buffer_pool
-rw-rw----. 1 mysql mysql 12582912 Nov  5 20:17 ibdata1
-rw-rw----. 1 mysql mysql 50331648 Nov  5 20:17 ib_logfile0
-rw-rw----. 1 mysql mysql 50331648 Nov  5 20:17 ib_logfile1
drwx------. 2 mysql mysql     4096 Nov  5 20:17 mysql
drwx------. 2 mysql mysql       20 Nov  5 20:17 performance_schema
drwx------. 2 mysql mysql       20 Nov  5 20:17 test


```

### 3.1.3 拷贝mysql配置文件

```
[root@node1 mysql]# mkdir /etc/mysql
[root@node1 mysql]# cp support-files/my-huge.cnf /etc/mysql/my.cnf
[root@node1 mysql]# 
[root@node1 mysql]# vim /etc/mysql/my.cnf
加入
datadir=/data/mysql


```

### 3.1.4 加入服务脚本

```
[root@node1 mysql]# cp support-files/mysql.server  /etc/init.d/mysqld
[root@node1 mysql]# chkconfig --add mysqld
[root@node1 mysql]# service mysqld start
Starting mysqld (via systemctl):                           [  OK  ]
[root@node1 mysql]# lsof -i :3306 
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
mysqld  2866 mysql   20u  IPv6  34354      0t0  TCP *:mysql (LISTEN)
```

### 3.1.5 导入环境变量并安全加固

```
[root@node1 mysql]# echo 'PATH=/usr/local/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
[root@node1 mysql]# source /etc/profile.d/mysql.sh
[root@node1 mysql]# mysql
[root@node1 mysql]# /usr/local/mysql/bin/mysql_secure_installation 
```

## 4.1.1 mysql源码编译安装

```
准备环境
 yum -y install bison bison-devel zlib-devel libcurl-devel libarchive-devel boost-devel
 yum -y install gcc gcc-c++ cmake ncurses-devel gnutls-devel libxml2-devel
 yum -y install openssl-devel libevent-devel libaio-devel


```

## 4.1.2 准备用户和数据目录

```
mkdir /data/mysql/ -p
useradd -r -s /sbin/nologin -d /data/mysql/ mysql
chown mysql.mysql /data/mysql
tar xf mariadb-10.2.25.tar.gz 

```

```
 cmake . \
 -DCMAKE_INSTALL_PREFIX=/app/mysql \
 -DMYSQL_DATADIR=/data/mysql/ \
 -DSYSCONFDIR=/etc/ \
 -DMYSQL_USER=mysql \
 -DWITH_INNOBASE_STORAGE_ENGINE=1 \
 -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
 -DWITH_PARTITION_STORAGE_ENGINE=1 \
 -DWITHOUT_MROONGA_STORAGE_ENGINE=1 \
 -DWITH_DEBUG=0 \
 -DWITH_READLINE=1 \
 -DWITH_SSL=system \
 -DWITH_ZLIB=system \
 -DWITH_LIBWRAP=0 \
 -DENABLED_LOCAL_INFILE=1 \
 -DMYSQL_UNIX_ADDR=/data/mysql/mysql.sock \
 -DDEFAULT_CHARSET=utf8 \
 -DDEFAULT_COLLATION=utf8_general_ci

```

## 5.1.1 创建多实例数据库

```
[root@node3 ~]# yum -y install mariadb-server
[root@node3 ~]# systemctl start mariadb
[root@node3 ~]# ls /var/lib/mysql/
aria_log.00000001  aria_log_control  ibdata1  ib_logfile0  ib_logfile1  mysql  mysql.sock  performance_schema  test
[root@node3 ~]# mkdir /mysql/{3306,3307,3308}/{data,bin,log,socket,pid,etc} -p
[root@node3 ~]# systemctl stop mariadb


```

### 5.1.2 生成数据文件

```
[root@node3 ~]# mysql_install_db --data=/mysql/3306/data --user=mysql
[root@node3 ~]# mysql_install_db --data=/mysql/3307/data --user=mysql
[root@node3 ~]# mysql_install_db --data=/mysql/3308/data --user=mysql
[root@node3 ~]# chown -R mysql.mysql /mysql/
[root@node3 ~]# ll /mysql/
total 0
drwxr-xr-x. 8 mysql mysql 76 Nov  5 21:31 3306
drwxr-xr-x. 8 mysql mysql 76 Nov  5 21:31 3307
drwxr-xr-x. 8 mysql mysql 76 Nov  5 21:31 3308
```

### 拷贝配置文件

```
[root@node3 ~]# cp /etc/my.cnf /mysql/3306/etc/
[root@node3 ~]# grep -vE "^$|#" /mysql/3306/etc/my.cnf
[mysqld]
port=3306
datadir=/mysql/3306/data
socket=/mysql/3306/socket/mysql.sock
symbolic-links=0
[mysqld_safe]
log-error=/mysql/3306/log/mariadb.log
pid-file=/mysql/3306/pid/mariadb.pid

[root@node3 ~]# cp /mysql/3306/etc/my.cnf /mysql/3307/etc/
[root@node3 ~]# cp /mysql/3306/etc/my.cnf /mysql/3308/etc/

```

### 5.1.3 修改端口号

```
[root@node3 ~]# sed -i 's/3306/3307/g' /mysql/3307/etc/my.cnf 
[root@node3 ~]# sed -i 's/3306/3308/g' /mysql/3308/etc/my.cnf 

```

### 5.1.4 准备脚本文件

```
[root@node3 bin]# cat mysqld 
#!/bin/bash
#chkconfig: 345 80 2
port=3306
mysql_user="root"
mysql_pwd="shaoxh"
cmd_path="/usr/bin"
mysql_basedir="/mysql"
mysql_sock="${mysql_basedir}/${port}/socket/mysql.sock"

function_start_mysql()
{
    if [ ! -e "$mysql_sock" ];then
      printf "Starting MySQL...\n"
      ${cmd_path}/mysqld_safe --defaults-file=${mysql_basedir}/${port}/etc/my.cnf  &> /dev/null  &
    else
      printf "MySQL is running...\n"
      exit
    fi
}


function_stop_mysql()
{
    if [ ! -e "$mysql_sock" ];then
       printf "MySQL is stopped...\n"
       exit
    else
       printf "Stoping MySQL...\n"
       ${cmd_path}/mysqladmin -u ${mysql_user} -p${mysql_pwd} -S ${mysql_sock} shutdown
   fi
}


function_restart_mysql()
{
    printf "Restarting MySQL...\n"
    function_stop_mysql
    sleep 2
    function_start_mysql
}

case $1 in
start)
    function_start_mysql
;;
stop)
    function_stop_mysql
;;
restart)
    function_restart_mysql
;;
*)
    printf "Usage: ${mysql_basedir}/${port}/bin/mysqld {start|stop|restart}\n"
esac

```

```
[root@node3 bin]# chmod +x mysqld 
[root@node3 bin]# cp mysqld /mysql/3307/bin/
[root@node3 bin]# cp mysqld /mysql/3308/bin/
[root@node3 bin]# sed -i 's/3306/3307/g' /mysql/3307/bin/mysqld 
[root@node3 bin]# sed -i 's/3306/3308/g' /mysql/3308/bin/mysqld 

```

### 5.1.5 启动mysql

```
[root@node3 bin]# /mysql/3306/bin/mysqld start
MySQL is running...
[root@node3 bin]# /mysql/3307/bin/mysqld start
Starting MySQL...
[root@node3 bin]# /mysql/3308/bin/mysqld start
Starting MySQL...
[root@node3 bin]# ss -tnl
State       Recv-Q Send-Q                           Local Address:Port                                          Peer Address:Port              
LISTEN      0      50                                           *:3307                                                     *:*                  
LISTEN      0      50                                           *:3308                                                     *:*                  
LISTEN      0      128                                          *:111                                                      *:*                  
LISTEN      0      5                                192.168.122.1:53                                                       *:*                  
LISTEN      0      128                                          *:22                                                       *:*                  
LISTEN      0      128                                  127.0.0.1:631                                                      *:*                  
LISTEN      0      100                                  127.0.0.1:25                                                       *:*                  
LISTEN      0      128                                  127.0.0.1:6010                                                     *:*                  
LISTEN      0      50                                           *:3306                                                     *:*                  
LISTEN      0      128                                         :::111                                                     :::*                  
LISTEN      0      128                                         :::22                                                      :::*                  
LISTEN      0      128                                        ::1:631                                                     :::*                  
LISTEN      0      100                                        ::1:25                                                      :::*                  
LISTEN      0      128                                        ::1:6010                                                    :::*                  
[root@node3 bin]# 

```

### 5.1.6 连接数据库

```
[root@node3 bin]# mysql -S /mysql/3306/socket/mysql.sock 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 5.5.65-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

```

### 5.1.7 设置密码关闭mysql

```
[root@node3 bin]# mysqladmin -S /mysql/3308/socket/mysql.sock password shaoxh
[root@node3 bin]# /mysql/3308/bin/mysqld stop -pshaoxh
Stoping MySQL...
[root@node3 bin]# 

```

## 6.1.1 创建数据库设置字符集

```
MariaDB [(none)]> create database test3 character set utf8mb4;

[root@node3 ~]# cd /mysql/3306/data/
[root@node3 test3]# cat db.opt 
default-character-set=utf8mb4
default-collation=utf8mb4_general_ci
[root@node3 test3]# 

```

### 6.1.2 修改数据库

```
MariaDB [(none)]> alter database test3 character set utf8;

[root@node3 test3]# cat db.opt 
default-character-set=utf8
default-collation=utf8_general_ci

```

###  6.1.3  创建表

```
MariaDB [test3]> create table student(id int unsigned auto_increment primary key,name varchar(20) not null, gender ENUM('m','f'
    -> ) default 'm',mobile char(11));
Query OK, 0 rows affected (0.00 sec)

MariaDB [test3]> desc student;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name   | varchar(20)      | NO   |     | NULL    |                |
| gender | enum('m','f')    | YES  |     | m       |                |
| mobile | char(11)         | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)


MariaDB [test3]> show create table student;
+---------+--------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------+| Table   | Create Table                                                                                                                         
                                                                                                      |+---------+--------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------+| student | CREATE TABLE `student` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `gender` enum('m','f') DEFAULT 'm',
  `mobile` char(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+---------+--------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------+1 row in set (0.00 sec)



```

### 6.1.4 插入数据

```
MariaDB [test3]> insert student(name,mobile) values ('tom','15201106074');
Query OK, 1 row affected (0.00 sec)

MariaDB [test3]> insert student(name,mobile) values ('jerry','15201116074');
Query OK, 1 row affected (0.00 sec)

MariaDB [test3]> select *  from student;
+----+-------+--------+-------------+
| id | name  | gender | mobile      |
+----+-------+--------+-------------+
|  1 | tom   | m      | 15201106074 |
|  2 | jerry | m      | 15201116074 |
+----+-------+--------+-------------+
2 rows in set (0.00 sec)


```

```
MariaDB [test3]> insert student(name,mobile,gender) values ('xiaoming','10086','m'),('xiaohong','10010','f');
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [test3]> select *  from student;
+----+----------+--------+-------------+
| id | name     | gender | mobile      |
+----+----------+--------+-------------+
|  1 | tom      | m      | 15201106074 |
|  2 | jerry    | m      | 15201116074 |
|  3 | xiaoming | m      | 10086       |
|  4 | xiaohong | f      | 10010       |
+----+----------+--------+-------------+
4 rows in set (0.00 sec)


```

```
MariaDB [test3]> insert student(name,mobile) values ('马哥','62985600');
Query OK, 1 row affected (0.00 sec)

MariaDB [test3]> select *  from student;
+----+----------+--------+-------------+
| id | name     | gender | mobile      |
+----+----------+--------+-------------+
|  1 | tom      | m      | 15201106074 |
|  2 | jerry    | m      | 15201116074 |
|  3 | xiaoming | m      | 10086       |
|  4 | xiaohong | f      | 10010       |
|  5 | 马哥     | m      | 62985600    |
+----+----------+--------+-------------+
5 rows in set (0.00 sec)


```

### 6.1.5 复制一个表结构

```
MariaDB [test3]> create table stu4 like student;
Query OK, 0 rows affected (0.00 sec)

MariaDB [test3]> show create table stu4
    -> ;
+-------+----------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------+| Table | Create Table                                                                                                                           
                                                                                                 |+-------+----------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------+| stu4  | CREATE TABLE `stu4` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `gender` enum('m','f') DEFAULT 'm',
  `mobile` char(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+----------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------+1 row in set (0.00 sec)


```

### 6.1.6 复制表结构和数据

```
MariaDB [test3]> create table stu1 select * from student;
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0
MariaDB [test3]> select * from stu1;
+----+----------+--------+-------------+
| id | name     | gender | mobile      |
+----+----------+--------+-------------+
|  1 | tom      | m      | 15201106074 |
|  2 | jerry    | m      | 15201116074 |
|  3 | xiaoming | m      | 10086       |
|  4 | xiaohong | f      | 10010       |
|  5 | 马哥     | m      | 62985600    |
+----+----------+--------+-------------+
5 rows in set (0.00 sec)


```

### 6.1.7 省略字段插入数据

```
MariaDB [test3]> select * from stu4;
Empty set (0.00 sec)

MariaDB [test3]> insert stu4 values (1,'gouzige','m','111111');
Query OK, 1 row affected (0.00 sec)

MariaDB [test3]> select * from stu4;
+----+---------+--------+--------+
| id | name    | gender | mobile |
+----+---------+--------+--------+
|  1 | gouzige | m      | 111111 |
+----+---------+--------+--------+
1 row in set (0.00 sec)

MariaDB [test3]> 

```

### 6.1.8  查看数据库字符级别

```

MariaDB [(none)]> show variables like "%character%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)


```

6.1.9 设置数据库的字符集

```
[root@node3 bin]# vim /mysql/3306/etc/my.cnf 
character_set_server=utf8mb4

```

6.2.1 设置客户端

```
[root@node3 data]# mysql -S /mysql/3306/socket/mysql.sock --default-character-set=utf8mb4  -pshaoxh 


MariaDB [(none)]> show variables like "%character%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

```

## 7.1.1 准备一个新的mysql

```
[root@node2 ~]# systemctl start mariadb
[root@node2 ~]# mysql < hellodb_innodb.sql 
[root@node2 ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 5.5.65-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

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

MariaDB [(none)]> use hellodb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [hellodb]> 

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
| toc               |
+-------------------+
7 rows in set (0.00 sec)

MariaDB [hellodb]> select * from students;
+-------+---------------+-----+--------+---------+-----------+
| StuID | Name          | Age | Gender | ClassID | TeacherID |
+-------+---------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu   |  22 | M      |       2 |         3 |
|     2 | Shi Potian    |  22 | M      |       1 |         7 |
|     3 | Xie Yanke     |  53 | M      |       2 |        16 |
|     4 | Ding Dian     |  32 | M      |       4 |         4 |
|     5 | Yu Yutong     |  26 | M      |       3 |         1 |
|     6 | Shi Qing      |  46 | M      |       5 |      NULL |
|     7 | Xi Ren        |  19 | F      |       3 |      NULL |
|     8 | Lin Daiyu     |  17 | F      |       7 |      NULL |
|     9 | Ren Yingying  |  20 | F      |       6 |      NULL |
|    10 | Yue Lingshan  |  19 | F      |       3 |      NULL |
|    11 | Yuan Chengzhi |  23 | M      |       6 |      NULL |
|    12 | Wen Qingqing  |  19 | F      |       1 |      NULL |
|    13 | Tian Boguang  |  33 | M      |       2 |      NULL |
|    14 | Lu Wushuang   |  17 | F      |       3 |      NULL |
|    15 | Duan Yu       |  19 | M      |       4 |      NULL |
|    16 | Xu Zhu        |  21 | M      |       1 |      NULL |
|    17 | Lin Chong     |  25 | M      |       4 |      NULL |
|    18 | Hua Rong      |  23 | M      |       7 |      NULL |
|    19 | Xue Baochai   |  18 | F      |       6 |      NULL |
|    20 | Diao Chan     |  19 | F      |       7 |      NULL |
|    21 | Huang Yueying |  22 | F      |       6 |      NULL |
|    22 | Xiao Qiao     |  20 | F      |       1 |      NULL |
|    23 | Ma Chao       |  23 | M      |       4 |      NULL |
|    24 | Xu Xian       |  27 | M      |    NULL |      NULL |
|    25 | Sun Dasheng   | 100 | M      |    NULL |      NULL |
+-------+---------------+-----+--------+---------+-----------+
25 rows in set (0.00 sec)


```

###  7.1.2 字段赋值

```
MariaDB [hellodb]> insert students set name="mage",age=30,gender='M';
Query OK, 1 row affected (0.00 sec)

```

### 7.1.3  另一张表的查询结果赋值

```
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

MariaDB [hellodb]> 
MariaDB [hellodb]> 
MariaDB [hellodb]> insert students(name,age) select name,age from teachers;
Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0



```

7.1.4  修改孙大圣的班级 

```
MariaDB [hellodb]> update students set ClassID=1 where StuID=25;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0


```

7.1.5 登录数据库加上 -U 防止 不加where修改信息

```
[root@node2 ~]# mysql -U

MariaDB [hellodb]> update students set ClassID=1 ;
ERROR 1175 (HY000): You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column

```

7.1.6 删除大于id大于20的用户

```
MariaDB [hellodb]> delete from students where StuID>20;
Query OK, 10 rows affected (0.00 sec)


```

