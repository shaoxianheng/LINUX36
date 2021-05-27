## LAMP应用

## 模块方式运行php

```
[root@node1 ~]# yum -y install httpd php php-mysql mariadb-server
[root@node1 ~]# vim /var/www/html/index.php
<?php
echo date("Y/m/d H:i:s");
phpinfo();
?>

[root@node1 ~]# systemctl start httpd

浏览器访问：
http://192.168.0.111

php代码：
[root@node1 ~]# cat /var/www/html/test1.php
<?php
echo "<h1> hello world!</h1>"
?>

[root@node1 ~]# cat /var/www/html/test2.php
<h1>
  <?php echo "hello world!" ?>
</h1>


```

## 以pdo方式连接mysql

```
[root@node1 ~]# systemctl start mariadb
[root@node1 ~]# cat /var/www/html/pdo1.php
<?php
$dsn='mysql:host=localhost;dbname=test';
$username='root';
$passwd='';
$dbh=new PDO($dsn,$username,$passwd);
var_dump($dbh);
?>

浏览器访问：
http://192.168.0.111/pdo1.php


[root@node1 ~]# cat /var/www/html/pdo2.php
<?php
try {
$user='root';
$pass='';
$dbh = new PDO('mysql:host=localhost;dbname=mysql', $user, $pass);
foreach($dbh->query('SELECT user,host from user') as $row) {
print_r($row);
}$dbh = null;
} catch (PDOException $e) {
print "Error!: " . $e->getMessage() . "<br/>";
die();
}
?>

```

## phpmyadmin

```
解压缩
[root@node1 data]# unzip phpMyAdmin-4.4.14.1-all-languages.zip
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# ls index.php 
index.php
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# ls config.sample.inc.php 
config.sample.inc.php
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# cp config.sample.inc.php config.inc.php 
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# vim config.inc.php
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# 
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# 
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# 
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# pwd
/data/phpMyAdmin-4.4.14.1-all-languages
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# ls /var/www/html/
index.php  pdo1.php  pdo2.php  test1.php  test2.php
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# mkdir /var/www/html/pma
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# pwd
/data/phpMyAdmin-4.4.14.1-all-languages
[root@node1 phpMyAdmin-4.4.14.1-all-languages]# mv * /var/www/html/pma/

[root@node1 data]# yum install php-mbstring -y
[root@node1 data]# rpm -ql php-mbstring
[root@node1 data]# systemctl restart httpd

浏览器访问：
http://192.168.0.111/pma

设置mysql的密码

[root@node1 data]# mysql -uwpuser -pshaoxh -h192.168.0.111



```

总结：

```
安装http php php-mysql  mariadb
下载phpmyadmin 压缩包
依赖关系php-mbstring
设置mysql的密码
浏览器访问ip/pma

```

## 分离lamp应用

```
111主机 httpd php php-mysql  wordpress-5.0.3-zh_CN.tar.gz
[root@node1 ~]# yum -y install httpd php php-mysql

启动http服务
[root@node1 ~]# systemctl start httpd

配置php文件
[root@node1 html]# grep Shanghai /etc/php.ini
date.timezone = Asia/Shanghai

配置index.php
[root@node1 html]# cat /var/www/html/index.php 
<?php
echo date("Y/m/d H:i:s");
phpinfo();
?>

浏览器访问：
http://192.168.0.111

112 主机 mysql 
安装数据库并且启动
[root@node2 ~]# yum -y install mariadb-server
[root@node2 ~]# systemctl start mariadb
[root@node2 ~]# mysql

创建wordpress 的数据库
MariaDB [(none)]> create database wpdb;
MariaDB [(none)]> grant all on wpdb.* to wpuser@'192.168.0.%' identified by 'shaoxh';
MariaDB [(none)]> flush privileges;

测试是否可以远程连接mysql
[root@node1 html]# mysql -uwpuser -pshaoxh -h192.168.0.112

解压wordpress包
[root@node1 data]# ls
wordpress-5.0.3-zh_CN.tar.gz
[root@node1 data]# tar xf wordpress-5.0.3-zh_CN.tar.gz 
[root@node1 data]# ls
wordpress  wordpress-5.0.3-zh_CN.tar.gz
[root@node1 data]# cd wordpress/      
[root@node1 wordpress]# cp wp-config-sample.php wp-config.php

修改配置文件
define('DB_NAME', 'wpdb');

/** MySQL数据库用户名 */
define('DB_USER', 'wpuser');

/** MySQL数据库密码 */
define('DB_PASSWORD', 'shaoxh');

/** MySQL主机 */
define('DB_HOST', '192.168.0.112');
[root@node1 wordpress]# mkdir /var/www/html/wordpress
[root@node1 wordpress]# mv * /var/www/html/wordpress

浏览器访问
http://192.168.0.111/wordpress/
或者
[root@node1 data]# mv wordpress /var/www/html/
[root@node1 data]# ls /var/www/html/
index.php  wordpress
[root@node1 data]# 
[root@node1 data]# 
[root@node1 data]# setfacl -m u:apache:rwx /var/www/html/wordpress/
撤销
[root@node1 data]# setfacl -Rb /var/www/html/wordpress/



```

## Discuz搭建

```
[root@node1 data]# ls
Discuz_X3.3_SC_UTF8.zip  readme  upload  utility  wordpress-5.0.3-zh_CN.tar.gz
[root@node1 data]# mv upload/ /var/www/html/forum
[root@node1 data]# setfacl -Rm u:apache:rwx /var/www/html/forum/

node2
创建数据库
MariaDB [(none)]> create database discuz;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all on discuz.* to dzuser@'192.168.0.%' identified by 'shaoxh';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;

```

## powerDNS

```
启用epel源
[root@node1 data]# yum -y install pdns pdns-backend-mysql

node2 创建数据库
MariaDB [(none)]> create database powerdns;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all on powerdns.* to 'powerdns'@'192.168.0.%' identified by 'shaoxh';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;

创建powerdns数据库中的表，参看下面文档实现
 https://doc.powerdns.com/md/authoritative/backend-generic-mysql/
 
 [root@node1 data]# vim /etc/pdns/pdns.conf 
launch=gmysql
gmysql-host=192.168.0.112
gmysql-port=3306
gmysql-dbname=powerdns
gmysql-user=powerdns
gmysql-password=shaoxh

[root@node1 data]# grep -vE "#|^$" /etc/pdns/pdns.conf
launch=gmysql
gmysql-host=192.168.0.112
gmysql-port=3306
gmysql-dbname=powerdns
gmysql-user=powerdns
gmysql-password=shaoxh
setgid=pdns
setuid=pdns

启动pdns
[root@localhost ~]# systemctl start pdns
[root@localhost ~]#  systemctl enable httpd
[root@localhost ~]# cd /var/www/html
[root@localhost html]# wget http://downloads.sourceforge.net/project/poweradmin/poweradmin-2.1.7.tgz
[root@localhost html]# tar xf poweradmin-2.1.7.tgz 
[root@localhost html]# ls
poweradmin-2.1.7  poweradmin-2.1.7.tgz
[root@localhost html]# mv poweradmin-2.1.7 poweradmin

```

## 启用php-xcache加速

```
[root@node1 ~]# yum -y install php-devel httpd php php-mysql
[root@node1 data]# tar xf xcache-3.2.0.tar.gz 
[root@node1 data]# ls
xcache-3.2.0  xcache-3.2.0.tar.gz
[root@node1 data]# cd xcache-3.2.0/

生成configure脚本
[root@node1 xcache-3.2.0]# phpize --clean && phpize
[root@node1 xcache-3.2.0]# ./configure --enable-xcache
[root@node1 xcache-3.2.0]# make && make install
[root@node1 xcache-3.2.0]# cp xcache.ini /etc/php.d/
[root@node1 xcache-3.2.0]# systemctl restart httpd.service




112安装mysql
[root@node2 ~]# yum -y install mariadb-server
MariaDB [(none)]> create database wpdb;
Query OK, 1 row affected (0.01 sec)

MariaDB [(none)]> grant all on wpdb.* to 'wpuser'@'192.168.0.%' identified by 'shaoxh';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

浏览器访问：
http://192.168.0.111/wordpress/

测试结果：
[root@node2 ~]# ab -c10 -n100 http://192.168.0.111/wordpress/
Requests per second:    18.84 [#/sec] (mean)

```

## 独立服务php-fpm (端口号)

```
111 安装httpd php-fpm php-mysql
[root@node1 ~]# yum -y install php-fpm httpd php-mysql
[root@node1 data]# tar xf wordpress-5.0.3-zh_CN.tar.gz 
[root@node1 data]# mv wordpress /var/www/html/
[root@node1 data]# setfacl -m u:apache:rwx /var/www/html/wordpress/

配置fastcgi
[root@node1 wordpress]# cat /etc/httpd/conf.d/fcgi.conf 
DirectoryIndex index.php
ProxyRequests Off
ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/var/www/html/$1
[root@node1 wordpress]# systemctlr start httpd php-fpm


112安装mysql
[root@node2 ~]# yum -y install mariadb-server
MariaDB [(none)]> create database wpdb;
Query OK, 1 row affected (0.01 sec)

MariaDB [(none)]> grant all on wpdb.* to 'wpuser'@'192.168.0.%' identified by 'shaoxh';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

浏览器访问：
http://192.168.0.111/wordpress/


测试结果：
[root@node2 ~]# ab -c10 -n100 http://192.168.0.111/wordpress/
Requests per second:    6.79 [#/sec] (mean)

```

![image-20210116232349928](C:\Users\localhost\AppData\Roaming\Typora\typora-user-images\image-20210116232349928.png)

![image-20210116232445741](C:\Users\localhost\AppData\Roaming\Typora\typora-user-images\image-20210116232445741.png)





## 编译安装httpd2.4(UDS)

```
准备相应的包
[root@node1 ~]# cd /data/
[root@node1 data]# ls
apr-1.7.0.tar.gz  apr-util-1.6.1.tar.bz2  httpd-2.4.39.tar.bz2
[root@node2 data]# for i in * ;do tar xf $i;done
[root@node1 data]# ls
apr-1.7.0  apr-1.7.0.tar.gz  apr-util-1.6.1  apr-util-1.6.1.tar.bz2  httpd-2.4.39  httpd-2.4.39.tar.bz2
[root@node2 data]# mv apr-1.7.0 httpd-2.4.39/srclib/apr
[root@node2 data]# mv apr-util-1.6.1 httpd-2.4.39/srclib/apr-util
[root@node2 httpd-2.4.39]# yum -y install gcc prce-devel openssl-devel expat-devel
./configure \
--prefix=/app/httpd24 \
--enable-so \
--enable-ssl \
--enable-cgi \
--enable-rewrite \
--with-zlib \
--with-pcre \
--with-included-apr \
--enable-modules=most \
--enable-mpms-shared=all \
--with-mpm=prefork
make && make install

[root@node1 httpd-2.4.39]# useradd -r apache 
[root@node1 httpd-2.4.39]# vim /app/httpd24/conf/httpd.conf 
User daemon
Group daemon
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
<IfModule dir_module>
    DirectoryIndex  index.php index.html
</IfModule>
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps
ProxyRequests Off
ProxyPassMatch "^/(.*\.php(/.*)?)$" "unix:/var/run/php5-fpm.sock|fcgi://localhost/app/httpd24/htdocs/"

安装php-fpm
[root@node1 httpd-2.4.39]# yum -y install php-fpm php-mysql
[root@node1 httpd-2.4.39]# vim /etc/php-fpm.d/www.conf 
修改下面几项
listen = /var/run/php5-fpm.sock
listen.owner = apache
listen.group = apache
listen.mode = 0666

上传wordpress
[root@node1 httpd-2.4.39]# cd /app/httpd24/htdocs/
[root@node1 htdocs]# tar xf wordpress-5.0.3-zh_CN.tar.gz 
[root@node1 htdocs]# setfacl -m u:apache:rwx wordpress/

启动：
[root@node1 httpd-2.4.39]# systemctl start php-fpm
[root@node1 httpd-2.4.39]# /app/httpd24/bin/apachectl start

node2 启动mysql
[root@node2 ~]# yum -y install mariadb-server
[root@node2 ~]# systemctl start mariadb

MariaDB [(none)]> create database wpdb;
Query OK, 1 row affected (0.01 sec)

MariaDB [(none)]> grant all on wpdb.* to 'wpuser'@'192.168.0.%' identified by 'shaoxh';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

浏览器访问：
http://192.168.0.111/wordpress/

测试结果
[root@node2 ~]# ab -c10 -n100 http://192.168.0.111/wordpress/
Requests per second:    7.42 [#/sec] (mean)



```

##  源码编译LAMP

```
112 主机安装mariadb
[root@node2 ~]# cd /data/
[root@node2 data]# ls
mariadb-10.2.25-linux-x86_64.tar.gz
[root@node2 data]# useradd -r -s /sbin/nologin -d /data/mysql mysql
[root@node2 data]# mkdir /data/mysql
[root@node2 data]# chown mysql.mysql /data/mysql
[root@node2 data]# tar xf mariadb-10.2.25-linux-x86_64.tar.gz -C /usr/local/
创建软链接
[root@node2 data]# cd /usr/local/
[root@node2 local]# ln -s mariadb-10.2.25-linux-x86_64/ mysql
[root@node2 local]# chown -R root.root mysql/
配置环境变量
[root@node2 local]# echo 'PATH=/usr/local/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
[root@node2 local]# . /etc/profile.d/mysql.sh
初始化脚本
[root@node2 local]# cd mysql/
[root@node2 mysql]# scripts/mysql_install_db --datadir=/data/mysql --user=mysql

拷贝配置文件
[root@node2 mysql]# cp -b support-files/my-huge.cnf /etc/my.cnf
修改配置文件
[mysqld] 
加入下面一行
datadir = /data/mysql

加入服务列表
[root@node2 mysql]# cp support-files/mysql.server /etc/init.d/mysqld
[root@node2 mysql]# chkconfig --add mysqld

启动mysql
[root@node2 mysql]# service mysqld start

创建数据库和远程连接用户
create database wordpress;
create database discuz;
grant all on wordpress.* to 'wordpress'@'192.168.0.%' identified by 'shaoxh';
grant all on discuz.* to 'discuz'@'192.168.0.%' identified by 'shaoxh';
flush privileges;

111 主机安装httpd php-fpm
[root@node1 data]# ll
total 44440
-rw-r--r--. 1 root root  1093896 Jan 14 23:12 apr-1.7.0.tar.gz
-rw-r--r--. 1 root root   428595 Jan 14 23:13 apr-util-1.6.1.tar.bz2
-rw-r--r--. 1 root root 10922155 Jan 15 20:29 Discuz_X3.3_SC_UTF8.zip
-rw-r--r--. 1 root root  7030539 Jan 14 23:12 httpd-2.4.39.tar.bz2
-rw-r--r--. 1 root root 14820606 Jan 16 23:01 php-7.3.5.tar.bz2
-rw-r--r--. 1 root root 11195223 Jan 16 23:02 wordpress-5.2.tar.gz
[root@node1 data]# tar xf apr-1.7.0.tar.gz 
[root@node1 data]# tar xf apr-util-1.6.1.tar.bz2 
[root@node1 data]# tar xf httpd-2.4.39.tar.bz2 
[root@node1 data]# mv apr-1.7.0 httpd-2.4.39/srclib/apr
[root@node1 data]# mv apr-util-1.6.1 httpd-2.4.39/srclib/apr-util
useradd -s /sbin/nologin -r apache
yum -y install gcc prce-devel openssl-devel expat-devel
./configure \
--prefix=/app/httpd24 \
--enable-so \
--enable-ssl \
--enable-cgi \
--enable-rewrite \
--with-zlib \
--with-pcre \
--with-included-apr \
--enable-modules=most \
--enable-mpms-shared=all \
--with-mpm=prefork
make && make install

加入环境变量
[root@node1 httpd-2.4.39]#echo 'PATH=/app/httpd24/bin/:$PATH' > /etc/profile.d/httpd24.sh
[root@node1 httpd-2.4.39]#. /etc/profile.d/httpd24.sh
[root@node1 httpd24]# apachectl start 


 配置httpd支持php
vim /app/httpd24/conf/httpd.conf
取消下面两行的注释
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
修改下面行
<IfModule dir_module>
 DirectoryIndex index.php index.html
</IfModule>
加下面几行
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps

<Virtualhost *:80>
Servername blog.magedu.com
DocumentRoot /data/wordpress
<Directory /data/wordpress>
require all granted
</Directory>
ProxyRequests Off
ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/data/wordpress/$1
</Virtualhost>


<Virtualhost *:80>
Servername forum.magedu.com
DocumentRoot /data/discuz
<Directory /data/discuz>
require all granted
</Directory>
ProxyRequests Off
ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/data/discuz/$1
</Virtualhost>
[root@node1 httpd24]# mkdir /data/wordpress
[root@node1 httpd24]# mkdir /data/discuz

安装epel源
[root@node1 httpd24]# yum -y install epel*
[root@node1 httpd24]# yum install libxml2-devel bzip2-devel libmcrypt-devel -y
[root@node1 data]# tar xf php-7.3.5.tar.bz2 
[root@node1 data]# cd php-7.3.5/
./configure --prefix=/app/php \
--enable-mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-openssl \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib \
--with-libxml-dir=/usr \
--with-config-file-path=/etc \
--with-config-file-scan-dir=/etc/php.d \
--enable-mbstring \
--enable-xml \
--enable-sockets \
--enable-fpm \
--enable-maintainer-zts \
--disable-fileinfo
make && make install
[root@node1 php-7.3.5]# cp php.ini-production /etc/php.ini
[root@node1 php-7.3.5]# cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
[root@node1 php-7.3.5]# chmod +x /etc/init.d/php-fpm
[root@node1 php-7.3.5]# chkconfig --add php-fpm
[root@node1 php-7.3.5]# chkconfig php-fpm on

[root@node1 php-7.3.5]# cd /app/php/etc
[root@node1 etc]# cp php-fpm.conf.default php-fpm.conf
[root@node1 etc]# cp php-fpm.d/www.conf.default php-fpm.d/www.conf
修改一下用户
user = apache
group = apache

[root@node1 etc]# service php-fpm start


[root@node1 php-7.3.5]# tar xf /data/wordpress-5.2.tar.gz -C /data/
[root@node1 data]# unzip Discuz_X3.3_SC_UTF8.zip 
[root@node1 data]# mv upload/* discuz/

设置权限
[root@node1 data]# setfacl -Rm u:apache:rwx discuz/ wordpress/

浏览器访问：

http://blog.magedu.com
http://forum.magedu.com/

测试结果：
[root@node1 data]# ab -c10 -n100 http://blog.magedu.com/wp-admin/
Requests per second:    9.17 [#/sec] (mean)

[root@node1 data]# ab -c10 -n100 http://forum.magedu.com/forum.php
Requests per second:    60.74 [#/sec] (mean)



```