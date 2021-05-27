

































# ..httpd知识点

1.1.1 MIME 类型

```
[root@node1 ~]# yum -y install mailcap
[root@node1 ~]# cat /etc/mime.types
```

## 查看外网的ip地址

```
[root@node1 ~]# curl ipinfo.io/ip
223.98.66.107

```

## 安装httpd服务

```
      
```

## 访问主页面

```
[root@node1 conf]# curl -I 192.168.0.111 
HTTP/1.1 403 Forbidden
Date: Sat, 26 Dec 2020 04:02:19 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Thu, 16 Oct 2014 13:20:58 GMT
ETag: "1321-5058a1e728280"
Accept-Ranges: bytes
Content-Length: 4897
Content-Type: text/html; charset=UTF-8

[root@node1 conf]# cat > /var/www/html/index.html
<h1>www.magedu.com</h1>
^C
[root@node1 conf]# !curl
curl -I 192.168.0.111 
HTTP/1.1 200 OK
Date: Sat, 26 Dec 2020 04:05:54 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Sat, 26 Dec 2020 04:05:42 GMT
ETag: "18-5b7562795f5c6"
Accept-Ranges: bytes
Content-Length: 24
Content-Type: text/html; charset=UTF-8

[root@node1 conf]# curl  192.168.0.111 
<h1>www.magedu.com</h1>

```

## 安全信息设置

```
[root@node1 httpd]# cat /etc/httpd/conf.d/test.conf
Servertokens prod
[root@node1 httpd]# curl -I 192.168.0.111 
HTTP/1.1 200 OK
Date: Sat, 26 Dec 2020 04:12:14 GMT
Server: Apache
Last-Modified: Sat, 26 Dec 2020 04:05:42 GMT
ETag: "18-5b7562795f5c6"
Accept-Ranges: bytes
Content-Length: 24
Content-Type: text/html; charset=UTF-8


```

```
[root@node1 httpd]# cp /etc/fstab /var/www/html/test.txt
[root@node1 httpd]# curl -I 192.168.0.111/test.txt
HTTP/1.1 200 OK
Date: Sat, 26 Dec 2020 04:16:16 GMT
Server: Apache
Last-Modified: Sat, 26 Dec 2020 04:16:04 GMT
ETag: "1d1-5b7564ca0b3ac"
Accept-Ranges: bytes
Content-Length: 465
Content-Type: text/plain; charset=UTF-8

[root@node1 httpd]# curl 192.168.0.111/test.txt

#
# /etc/fstab
# Created by anaconda on Sun Nov  1 09:18:03 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=7437e5ca-771f-42e2-aa98-0c6b894f4d05 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0

```

## 验证持久连接

```
[root@node1 httpd]# telnet 192.168.0.111 80
Trying 192.168.0.111...
Connected to 192.168.0.111.
Escape character is '^]'.
GET /index.html HTTP/1.1
host: 1.1.1.1

HTTP/1.1 200 OK
Date: Sat, 26 Dec 2020 04:25:50 GMT
Server: Apache
Last-Modified: Sat, 26 Dec 2020 04:05:42 GMT
ETag: "18-5b7562795f5c6"
Accept-Ranges: bytes
Content-Length: 24
Content-Type: text/html; charset=UTF-8

<h1>www.magedu.com</h1>

```

## 调试持久连接

```
[root@node1 httpd]# cat /etc/httpd/conf.d/test.conf
Servertokens prod
KeepAliveTimeout 20 

```

## httpd -M 查看加载的模块

```
[root@node1 ~]# httpd -M

```

## DSO 加载动态模块不需要重启

```
[root@node1 ~]# vim /etc/httpd/conf.modules.d/00-base.conf 

```

## 三种运行模型

```
[root@node1 conf.modules.d]# vim 00-mpm.conf

```

## 测试event模型的访问资源时间

```
[root@node1 conf.modules.d]# systemctl restart httpd
[root@node1 conf.modules.d]# ll /var/log/messages 
-rw-------. 1 root root 702761 Dec 27 22:47 /var/log/messages
[root@node1 conf.modules.d]# cp /var/log/messages /var/www/html/test.txt
cp: overwrite ‘/var/www/html/test.txt’? y
[root@node1 conf.modules.d]# chmod 644 /var/www/html/test.txt

[root@node2 ~]# ab -c 1000 -n 2000 http://192.168.0.111/test.txt

```

## prefork配置

```
[root@node1 ~]# cat /etc/httpd/conf.d/test.conf 
Servertokens prod
KeepAliveTimeout 20
StartServers 2000
MinSpareServers 2000
MaxSpareServers 2000
ServerLimit 2560
MaxRequestsPerChild 40000
MaxClients 2560


[root@node1 ~]# pstree -p|grep httpd |wc -l
2000
[root@node1 ~]# 
[root@node1 ~]# 
[root@node1 ~]# 
[root@node1 ~]# 
[root@node1 ~]# 
[root@node1 ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           5.7G        1.6G        3.4G        9.8M        743M        3.5G
Swap:          2.0G          0B        2.0G

```

## 明确授权目录文件

```
[root@node1 ~]# cat /etc/httpd/conf.d/test.conf 
DocumentRoot "/data/html"
<Directory "/data/html/">
    AllowOverride None
    Require all granted
</Directory>
[root@node1 ~]# systemctl restart httpd
[root@node1 ~]# curl http://192.168.0.111
shaoxh

[root@node1 ~]# cd /data/html/
[root@node1 html]# mkdir news
[root@node1 html]# echo news shaoxh > news/index.html
[root@node1 html]# curl http://192.168.0.111/news/
news shaoxh

子目录的软链接
[root@node1 html]# mkdir -pv /app/sports
mkdir: created directory ‘/app’
mkdir: created directory ‘/app/sports’
[root@node1 html]# echo shaoxh_sport > /app/sports/index.html
[root@node1 html]# ln -sv /app/sports/index.html /data/html/sports
‘/data/html/sports’ -> ‘/app/sports/index.html’
[root@node1 html]# curl http://192.168.0.111/sports
shaoxh_sport


```

## 默认页面

```
[root@node1 conf.d]# vim /etc/httpd/conf/httpd.conf
x
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>

```

## 默认错误页面

```
[root@node1 conf.d]# vim welcome.conf 

<LocationMatch "^/+$">
    Options -Indexes
    ErrorDocument 403 /.noindex.html
</LocationMatch>

```

## 显示目录结构和链接

```
[root@node1 conf.d]# mv welcome.conf{,.bak}
[root@node1 conf.d]# ls
autoindex.conf  README  test.conf  userdir.conf  welcome.conf.bak
[root@node1 conf.d]# cat /etc/httpd/conf.d/test.conf 
DocumentRoot "/data/html"
<Directory "/data/html/">
    AllowOverride None
    Require all granted
    options Indexes FollowSymLinks
</Directory>

```

## htaccess文件

```
需要授权
[root@node1 html]# cat /etc/httpd/conf.d/test.conf
DocumentRoot "/data/html"
<Directory "/data/html/">
    AllowOverride ALL
    Require all granted
    #options Indexes FollowSymLinks
</Directory>
[root@node1 html]# pwd
/data/html
[root@node1 html]# cat .htaccess 
options Indexes FollowSymLinks
[root@node1 html]# systemctl restart httpd

浏览器访问：192.168.0.111
```

## 拒绝文件访问

```
[root@node1 html]# vim /etc/httpd/conf.d/test.conf
DocumentRoot "/data/html"
<Directory "/data/html/">
    AllowOverride ALL
    Require all granted
    #options Indexes FollowSymLinks
</Directory>
<Files "*.conf">
    Require all denied    拒绝以conf结尾的文件
</Files>

[root@node1 html]# cat test.conf 
shaoconfig


```

## 正则表达式拒绝访问方式

```
[root@node1 html]# cat /etc/httpd/conf.d/test.conf 
......
<Filesmatch "\.conf$">
    Require all denied
</Filesmatch>

```

## 拒绝某个ip地址访问

```
[root@node1 html]# mkdir beijing
[root@node1 html]# echo beijing > beijing/index.html
[root@node1 html]# !vim
vim /etc/httpd/conf.d/test.conf 
......
<Directory "/data/html/beijing">
<requireall>
Require all granted
Require not ip 192.168.0.112
</requireall>
</Directory>

[root@node1 html]# curl http://192.168.0.111/beijing/index.html
beijing

[root@node2 ~]# curl http://192.168.0.111/beijing/index.html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /beijing/index.html
on this server.</p>
</body></html>

```

## 修改Servername

```
[root@node1 html]# cat /etc/httpd/conf.d/test.conf 
Servername test.magedu.com
```

## 跳转页面

```
[root@node1 html]# cat test1.html 
<html>
<head>
<title>html语言</title>
</head>
<body>
<p><a href=http://www.magedu.com>link<a>欢迎你</p>
</body>
</html>

```

## 引用另一台的图片

```
[root@node2 ~]# cat /var/www/html/test.html 
<img src="http://192.168.0.111/sunflower.jpg" >
[root@node1 html]# pwd
/data/html
[root@node1 html]# ll sunflower.jpg 
-rw-r--r--. 1 root root 4105 Dec 29 22:58 sunflower.jpg

```

## 查看字符级

```
[root@node1 html]# iconv -l
[root@node2 ~]# curl -I localhost
HTTP/1.1 403 Forbidden
Date: Wed, 30 Dec 2020 04:30:47 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Thu, 16 Oct 2014 13:20:58 GMT
ETag: "1321-5058a1e728280"
Accept-Ranges: bytes
Content-Length: 4897
Content-Type: text/html; charset=UTF-8

[root@node2 ~]# 

```

## alias别名

```
[root@node1 html]# mkdir /app/forum -p
[root@node1 html]# echo /app/forum/index.html > /app/forum/index.html
[root@node1 html]# cat /etc/httpd/conf.d/test.conf 
alias /bbs/ /app/forum/
<Directory "/app/forum">
    require all granted
</Directory>
[root@node1 html]# systemctl restart httpd
[root@node1 html]# curl http://192.168.0.111/bbs/
/app/forum/index.html

```

## 身份验证

```
[root@node1 html]# httpd -M |grep auth_basic
 auth_basic_module (shared)
 
 [root@node1 html]# mkdir admin
[root@node1 html]# echo /data/html/admin/index.html > admin/index.html
[root@node1 html]# cat admin/index.html 
/data/html/admin/index.html
[root@node1 html]# curl http://192.168.0.111
<h1> www.magedu.com </h1>
[root@node1 html]# curl http://192.168.0.111/admin/
/data/html/admin/index.html

创建虚拟用户
[root@node1 conf.d]# htpasswd -c .httpuser bob
New password: 
Re-type new password: 
Adding password for user bob
[root@node1 conf.d]# htpasswd  .httpuser alice
New password: 
Re-type new password: 
Adding password for user alice
[root@node1 conf.d]# cat .httpuser 
bob:$apr1$suRPjGn4$sMz56LSiHNGDDpyTs3Ipc1
alice:$apr1$3J4aaUle$thWBSlLkVSnV2k7HDCD2D.

配置虚拟用户目录认证
[root@node1 conf.d]# cat test.conf 
<Directory "/data/html/admin">
    AuthType Basic
    AuthName "admin Page"
    AuthUserFile "/etc/httpd/conf.d/.httpuser"
    Require user alice
</Directory>

重启服务
[root@node1 conf.d]# systemctl restart httpd

浏览器访问
http://192.168.0.111/admin/

```

## 配置文件valid-user 

```
<Directory "/data/html/admin">
    AuthType Basic
    AuthName "admin Page"
    AuthUserFile "/etc/httpd/conf.d/.httpuser"
    Require valid-user   文件里所有用户都有效
</Directory>

```

## 创建htaccess文件

```
[root@node1 admin]# cat .htaccess 
AuthType Basic
AuthName "Admin Page"
AuthUserFile "/etc/httpd/conf.d/.httpuser"
Require valid-user

[root@node1 admin]# cat /etc/httpd/conf.d/test.conf
<Directory "/data/html/admin">
    allowOverride authconfig
</Directory>

```

## 基于组账号验证

```
[root@node1 conf.d]# cat .httpgroup
g1: alice bob
g2: jack rose

创建虚拟用户
[root@node1 conf.d]# htpasswd .httpuser jack
New password: 
Re-type new password: 
Adding password for user jack
[root@node1 conf.d]# htpasswd .httpuser rose

[root@node1 admin]# cat .htaccess 
AuthType Basic
AuthName "Admin Page"
AuthUserFile "/etc/httpd/conf.d/.httpuser"
AuthGroupFile "/etc/httpd/conf.d/.httpgroup"
Require group g2
[root@node1 admin]# !sys
systemctl restart httpd

浏览器访问
http://192.168.0.111/admin/

```

## 账号家目录共享

```
[root@node1 admin]# httpd -M |grep user
 authz_user_module (shared)
 userdir_module (shared)

创建需要的目录和文件
[root@node1 conf.d]# mkdir /home/shaoxh/public_html
[root@node1 conf.d]# echo shaoxh > /home/shaoxh/public_html/index.html

编辑配置文件
[root@node1 conf.d]# grep -vE "^$|#" userdir.conf
<IfModule mod_userdir.c>
    #UserDir disabled
    UserDir public_html
</IfModule>
<Directory /home/shaoxh/public_html>
    require all granted
</Directory>

加上执行权限
[root@node1 conf.d]# setfacl -m u:apache:x /home/shaoxh/

浏览器访问：
http://192.168.0.111/~shaoxh

使用验证功能

[root@node1 html]# vim /etc/httpd/conf.d/userdir.conf 
.......
<Directory "/home/shaoxh/public_html">
    AuthType Basic
    AuthName "shaoxh page"
    AuthUserFile "/etc/httpd/conf.d/.httpuser"
    Require valid-user
</Directory>
......

```

## status_mode

```
[root@node1 html]# httpd -M |grep status
 status_module (shared)
[root@node1 html]# 
[root@node1 html]# vim /etc/httpd/conf.d/test.conf 
<Location "/status">
SetHandler server-status
</Location>

浏览器访问：
http://192.168.0.111/status

限制ip访问
[root@node1 html]# vim /etc/httpd/conf.d/test.conf 
<Location "/status">
SetHandler server-status
<requireany>
require all denied
require ip 192.168.0.0/24
</requireany>
</Location>

```

## 虚拟主机

```
创建目录
[root@node1 html]# cd /data/
[root@node1 data]# mkdir {a,b,c}site
[root@node1 data]# ls
asite  bsite  csite  html
[root@node1 data]# echo www.a.com > asite/index.html
[root@node1 data]# echo www.b.com > bsite/index.html
[root@node1 data]# echo www.c.com > csite/index.html

增加ip地址
[root@node1 data]# ip addr a 192.168.0.210/24 dev ens33
[root@node1 data]# ip addr a 192.168.0.211/24 dev ens33

[root@node1 data]# vim /etc/httpd/conf.d/test.conf 
<Virtualhost 192.168.0.111:80>
DocumentRoot /data/asite
<Directory "/data/asite">
Require all granted
</Directory>
</Virtualhost>
<Virtualhost 192.168.0.210:80>
DocumentRoot /data/bsite
<Directory "/data/bsite">
Require all granted
</Directory>
</Virtualhost>

<Virtualhost 192.168.0.211:80>
DocumentRoot /data/csite
<Directory "/data/csite">
Require all granted
</Directory>
</Virtualhost>

[root@node1 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.111 www.a.com
192.168.0.210 www.b.com
192.168.0.211 www.c.com

[root@node1 ~]# curl www.a.com
www.a.com
[root@node1 ~]# curl www.b.com
www.b.com
[root@node1 ~]# 
[root@node1 ~]# curl www.c.com
www.c.com


```

## 基于端口号访问

```
[root@node1 ~]# cat /etc/httpd/conf.d/test.conf
Listen 81
Listen 82
Listen 83
<Virtualhost *:81>
DocumentRoot /data/asite
<Directory "/data/asite">
Require all granted
</Directory>
</Virtualhost>
<Virtualhost *:83>
DocumentRoot /data/bsite
<Directory "/data/bsite">
Require all granted
</Directory>
</Virtualhost>

<Virtualhost *:82>
DocumentRoot /data/csite
<Directory "/data/csite">
Require all granted
</Directory>
</Virtualhost>

```

## 基于域名访问

```
[root@node1 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.111 www.a.com www.b.com  www.c.com
[root@node1 ~]# cat /etc/httpd/conf.d/test.conf 
<Virtualhost *:80>
DocumentRoot /data/asite
ServerName www.a.com
<Directory "/data/asite">
Require all granted
</Directory>
</Virtualhost>
<Virtualhost *:80>
DocumentRoot /data/bsite
ServerName www.b.com
<Directory "/data/bsite">
Require all granted
</Directory>
</Virtualhost>

<Virtualhost *:80>
DocumentRoot /data/csite
ServerName www.c.com
<Directory "/data/csite">
Require all granted
</Directory>
</Virtualhost>

使用telnet测试
[root@node1 ~]# telnet www.a.com 80
Trying 192.168.0.111...
Connected to www.a.com.
Escape character is '^]'.
GET / HTTP/1.1 
host: www.c.com

HTTP/1.1 200 OK
Date: Mon, 04 Jan 2021 08:04:20 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Mon, 04 Jan 2021 02:54:21 GMT
ETag: "a-5b80a34f30110"
Accept-Ranges: bytes
Content-Length: 10
Content-Type: text/html; charset=UTF-8

www.c.com
Connection closed by foreign host.

```

## mod_deflate

```
[root@node1 ~]# httpd -M |grep deflate
 deflate_module (shared)
[root@node1 conf.modules.d]# grep deflate 00-base.conf
LoadModule deflate_module modules/mod_deflate.so

准备访问页面
 cp /var/log/messages asite/m.txt
 chmod a+r asite/m.txt
 
 添加压缩功能
 [root@node1 data]# tail -4 /etc/httpd/conf.d/test.conf
AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
DeflateCompressionLevel 9

重启服务
systemctl restart httpd

浏览器访问：
http://192.168.0.111/m.txt

调试功能f12打开
Content-Encoding: gzip

[root@node1 data]# curl -I --compressed 192.168.0.111/m.txt
HTTP/1.1 200 OK
Date: Mon, 04 Jan 2021 08:36:53 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Mon, 04 Jan 2021 08:23:04 GMT
ETag: "aa58e-5b80ecc9536ca-gzip"
Accept-Ranges: bytes
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Type: text/plain; charset=UTF-8

```

## https功能

```
[root@node1 data]# yum -y install mod_ssl
[root@node1 data]# httpd -M |grep ssl
 ssl_module (shared)
 [root@node1 data]# rpm -ql mod_ssl
/etc/httpd/conf.d/ssl.conf
/etc/httpd/conf.modules.d/00-ssl.conf
/usr/lib64/httpd/modules/mod_ssl.so
/usr/libexec/httpd-ssl-pass-dialog
/var/cache/httpd/ssl
[root@node1 data]# systemctl restart httpd

[root@node1 ~]# ss -tnl
     80
     443
浏览器访问：
https://


```

## 创建私有CA实现https

```
CA创建
[root@node1 CA]# (umask 077; openssl genrsa -out private/cakey.pem 4096)
Generating RSA private key, 4096 bit long modulus
..................................................................................................++
...............................................++
e is 65537 (0x10001)

[root@node1 CA]# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:beijing
Locality Name (eg, city) [Default City]:beijing
Organization Name (eg, company) [Default Company Ltd]:magedu
Organizational Unit Name (eg, section) []:devops
Common Name (eg, your name or your server's hostname) []:ca.magedu.com
Email Address []:admin@magedu.com

[root@node1 CA]# touch /etc/pki/CA/index.txt
[root@node1 CA]# echo 01 > /etc/pki/CA/serial

申请证书：
[root@node1 conf.d]# mkdir ssl
[root@node1 conf.d]# cd ssl/
[root@node1 ssl]# pwd
/etc/httpd/conf.d/ssl
[root@node1 ssl]# (umask 066;openssl genrsa -out httpd.key)
[root@node1 ssl]# openssl req -new -key httpd.key -out httpd.csr

颁发证书：

[root@node1 ssl]# openssl ca -in httpd.csr -out httpd.crt -days 100
[root@node1 ssl]# cp /etc/pki/CA/cacert.pem .
[root@node1 ssl]# ll
total 12
-rw-r--r--. 1 root root 2114 Jan  4 04:53 cacert.pem
-rw-r--r--. 1 root root    0 Jan  4 04:51 httpd.crt
-rw-r--r--. 1 root root 1005 Jan  4 04:46 httpd.csr
-rw-------. 1 root root 1679 Jan  4 04:45 httpd.key

[root@node1 ssl]# grep SSLCertificate /etc/httpd/conf.d/ssl.conf
SSLCACertificateFile /etc/httpd/conf.d/ssl/cacert.pem
SSLCertificateFile /etc/httpd/conf.d/ssl/httpd.crt
SSLCertificateKeyFile /etc/httpd/conf.d/ssl/httpd.key


```

 

```
[root@node1 ssl]# curl --cacert cacert.pem https://www.a.com
[root@node1 ssl]# openssl s_client -connect www.a.com:443

```

## 301永久重定向

```
[root@node1 html]# curl -I http://www.360buy.com
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Tue, 05 Jan 2021 09:57:10 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: http://www.jd.com/

```

## 302临时重定向

```
[root@node1 html]# curl -I http://www.jd.com
HTTP/1.1 302 Moved Temporarily
Server: nginx
Date: Tue, 05 Jan 2021 09:57:27 GMT
Content-Type: text/html
Content-Length: 138
Connection: keep-alive
Location: https://www.jd.com/

```

## 测试永久重定向	

```
[root@node1 ~]# cat /etc/httpd/conf.d/test.conf
<Virtualhost *:80>
DocumentRoot /data/asite
ServerName www.a.com
<Directory "/data/asite">
Require all granted
</Directory>
redirect Permanent / http://www.b.com
</Virtualhost>
<Virtualhost *:80>
DocumentRoot /data/bsite
ServerName www.b.com
<Directory "/data/bsite">
Require all granted
</Directory>
</Virtualhost>

<Virtualhost *:80>
DocumentRoot /data/csite
ServerName www.c.com
<Directory "/data/csite">
Require all granted
</Directory>
</Virtualhost>
ServerName www.magedu.com

AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
DeflateCompressionLevel 9

[root@node1 ~]# curl -L http://www.a.com 
www.b.com


```

## 实现http到httpds的重定向

```
[root@node1 conf.d]# vim /etc/httpd/conf/httpd.conf 
DocumentRoot "/var/www/html"
#redirect temp / https://www.a.com
RewriteEngine on
RewriteRule ^(/.*)* https://%{HTTP_HOST}$1 [redirect=302]

[root@node1 conf.d]# curl https://www.a.com -kL
www.a.com

```

## HSTS

```
[root@node1 conf.d]# curl -I http://www.jd.com
HTTP/1.1 302 Moved Temporarily
Server: nginx
Date: Tue, 05 Jan 2021 10:55:11 GMT
Content-Type: text/html
Content-Length: 138
Connection: keep-alive
Location: https://www.jd.com/
Access-Control-Allow-Origin: *
Timing-Allow-Origin: *
X-Trace: 302-1609844111875-0-0-0-0-0
Strict-Transport-Security: max-age=360

```

## 设置HSTS 时间

```
[root@node1 conf.d]# vim /etc/httpd/conf/httpd.conf
DocumentRoot "/var/www/html"
#redirect temp / https://www.a.com
RewriteEngine on
RewriteRule ^(/.*)* https://%{HTTP_HOST}$1 [redirect=302]
Header always set Strict-Transport-Security "max-age=31536000"

[root@node1 conf.d]# curl -I http://www.a.com
HTTP/1.1 302 Found
Date: Tue, 05 Jan 2021 11:01:52 GMT
Server: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips
Strict-Transport-Security: max-age=31536000
Location: https://www.a.com/
Content-Type: text/html; charset=iso-8859-1


```

## 反向代理

```
环境准备3台主机
反代服务器配置
[root@node2 ~]# cat /etc/httpd/conf.d/test.conf
ProxyPass "/" "http://192.168.0.111/"
ProxyPassReverse "/" "http://192.168.0.111"

real server配置
[root@node1 ~]# curl localhost
192.168.0.111 Real Server

客户端配置
[root@node3 ~]# curl http://192.168.0.112
192.168.0.111 Real Server

```

## 特定URL反向代理

```
real server
[root@node1 ~]# cp /usr/share/pixmaps/faces/sunflower.jpg /var/www/html/a.jpg

代理服务器
[root@node2 ~]# cat /etc/httpd/conf.d/test.conf 
ProxyPass "/images" "http://192.168.0.111/"
ProxyPassReverse "/images" "http://192.168.0.111"

客户端
浏览器访问：
http://192.168.0.112 不走代理
http://192.168.0.112/images/a.jpg  走代理
```

## 设置cookie

```
安装php
[root@node1 html]# yum -y install php
[root@node1 html]# cat setcookie.php 
<?php
setcookie('username','wang');
setcookie('title','cto',time()+3600);
phpinfo();
?>

[root@node1 html]# systemctl restart httpd

浏览器访问：
http://192.168.0.111/setcookie.php

```

## links 查看源代码

```
[root@node1 html]# links --source http://192.168.0.111
192.168.0.111 Real Server

```

## links下载页面

```
[root@node1 html]# links --dump http://www.magedu.com

```

## wget 静默标准输出

```
[root@node1 html]# wget -q -O - http://192.168.0.111
192.168.0.111 Real Server

[root@node1 html]# wget -O /root/a.txt http://192.168.0.111

指定下载后的名称

```

## 网站上执行脚本

```

[root@node1 html]# wget -q -O - http://192.168.0.111/a.sh|bash
hello
[root@node1 html]# cat a.sh 
#!/bash/bash
echo hello

```

## curl命令

```
模拟ie11浏览器
[root@node1 html]# curl -A IE11 http://192.168.0.111
192.168.0.111 Real Server
[root@node1 html]# tail -n 1 /var/log/httpd/access_log
192.168.0.111 - - [13/Jan/2021:02:33:44 -0500] "GET / HTTP/1.1" 200 26 "-" "IE11"

从百度跳转访问
[root@node1 html]# curl -e http://www.baidu.com http://192.168.0.111
192.168.0.111 Real Server
[root@node1 html]# tail -n 1 /var/log/httpd/access_log
192.168.0.111 - - [13/Jan/2021:02:35:38 -0500] "GET / HTTP/1.1" 200 26 "http://www.baidu.com" "curl/7.29.0"
```

## 日志滚动工具

```
rotatelogs 

```

## apachectl 

```
root@node1 html]# apachectl start


```

## CentOS 7 编译安装httpd-2.4

```
[root@node1 data]# ls
apr-1.7.0.tar.gz  apr-util-1.6.1.tar.bz2  httpd-2.4.39.tar.bz2
[root@node1 data]# tar xf apr-1.7.0.tar.gz 
[root@node1 data]# tar xf apr-util-1.6.1.tar.bz2 
[root@node1 data]# tar xf httpd-2.4.39.tar.bz2 

合并包
[root@node1 data]# cp -r apr-1.7.0 httpd-2.4.39/srclib/apr
[root@node1 data]# cp -r apr-util-1.6.1 httpd-2.4.39/srclib/apr-util

依赖的包
[root@node1 data]# yum -y install gcc pcre-devel openssl-devel expat-devel

编译安装
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

准备环境变量
[root@node1 httpd24]# echo 'PATH=/app/httpd24/bin:$PATH' > /etc/profile.d/httpd24.sh
[root@node1 httpd24]# . /etc/profile.d/httpd24.sh
[root@node1 httpd24]# cat /etc/profile.d/httpd24.sh 
PATH=/app/httpd24/bin:$PATH
[root@node1 httpd24]# echo $PATH
/app/httpd24/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
启动服务
[root@node1 httpd24]# apachectl start

[root@node1 httpd24]# curl localhost
<html><body><h1>It works!</h1></body></html>

编译安装的目录主页路径
[root@node1 htdocs]# cat /app/httpd24/htdocs/index.html
<html><body><h1>It works!</h1></body></html>

配置文件路径
[root@node1 conf]# ls /app/httpd24/conf/httpd.conf 
/app/httpd24/conf/httpd.conf

配置启动脚本
[root@node2 ~]# scp /usr/lib/systemd/system/httpd.service 192.168.0.111:/data
[root@node1 data]# vim httpd.service 
Type=forking
ExecStart=                -k start
[root@node1 data]# cp httpd.service /usr/lib/systemd/system/
[root@node1 data]# systemctl daemon-reload
[root@node1 data]# systemctl start httpd  

方法2：
[root@node1 ~]# cat /etc/rc.local 
/app/httpd24/bin/apachectl start
[root@node1 ~]# chmod +x /etc/rc.d/rc.local


·	

```

