# Nginx

## 1.1大型网站一般用到的是nginx

```
[root@node1 ~]# curl -I http://www.jd.com
HTTP/1.1 302 Moved Temporarily
Server: nginx
Date: Sat, 13 Feb 2021 01:11:13 GMT
Content-Type: text/html
Content-Length: 138
Connection: keep-alive
Location: https://www.jd.com/
Access-Control-Allow-Origin: *
Timing-Allow-Origin: *
X-Trace: 302-1613178673861-0-0-0-0-0
Strict-Transport-Security: max-age=360

```

## 1.2 I/O模型

```
阻塞型、非阻塞型、复用型、信号驱动型、异步
```

##  1.3 配置nginx的yum源

```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

## 1.4 配置nginx主页并且启动

```
[root@node1 yum.repos.d]# nginx  启动
[root@node1 yum.repos.d]# echo Welcome to magedu > /usr/share/nginx/html/index.html
[root@node1 yum.repos.d]# curl localhost 
Welcome to magedu

[root@node1 yum.repos.d]# nginx -h   帮助
[root@node1 yum.repos.d]# nginx -v   版本
nginx version: nginx/1.18.0
[root@node1 yum.repos.d]# nginx -V   编译的模块
[root@node1 yum.repos.d]# nginx -s  发送信号
stop  reload
[root@node1 yum.repos.d]# nginx -t   检查语法
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@node1 yum.repos.d]# nginx -T  显示配置项


```

## 1.5 源码编译nginx

```
下载nginx源码
[root@node1 src]# wget http://nginx.org/download/nginx-1.16.0.tar.gz
安装所依赖的包
[root@node1 src]# yum -y install pcre-devel openssl-devel zlib-devel
创建nginx用户
[root@node1 src]# useradd -r -s /sbin/nologin nginx

解压
[root@node1 src]# tar xf nginx-1.16.0.tar.gz 
[root@node1 src]# cd nginx-1.16.0/
编译
./configure --prefix=/apps/nginx \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-pcre \
--with-stream \
--with-stream_ssl_module \
--with-stream_realip_module
安装
make && make install 
```

## 1.6 创建软链接

```
[root@node1 nginx-1.16.0]# ln -s /apps/nginx/sbin/nginx /usr/sbin
启动nginx
[root@node1 nginx-1.16.0]# nginx 
[root@node1 nginx-1.16.0]# nginx -v  查看版本
nginx version: nginx/1.16.0

```

## 1.7 访问错误页面

```
[root@node1 nginx]# curl localhost/xxx
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.16.0</center>
</body>
</html>

```

## 1.8 nginx全局配置

```
正常运行必备的配置
user  nginx;
pid  /var/run/nginx.pid;
inculde file|mask;
load_module file;

性能优化相关的配置
worker_processes 1;
worker_cpu_affinity 0001;
worker_priority 19;
worker_rlimit_nofile 65535;

```

## 1.9 nginx 事件驱动相关的配置

```
worker_connections 10240;
use epoll;
accept_mutex on;
multi_accept on;
```

## 2.1 nginx 调试和定位问题的配置

```
daemon on;
master_process on;
error_log  /var/log/nginx/error.log warn;
```

## 2.2 配置字符集

```
[root@node2 ~]# cat /etc/nginx/conf.d/test.conf 
charset utf-8;

```

## 2.3 隐藏server版本

```
[root@node2 ~]# cat /etc/nginx/conf.d/test.conf 
charset utf-8;
server_tokens off;

[root@node2 ~]# !curl
curl -I localhost
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 16 Feb 2021 07:08:56 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 18
Last-Modified: Tue, 16 Feb 2021 01:09:14 GMT
Connection: keep-alive
ETag: "602b1b3a-12"
Accept-Ranges: bytes

```

## 2.4 自定义nginx版本信息

```
如果想自定义响应报文的nginx版本信息，需要修改源码文件，重新编译
如果server_tokens on，修改 src/core/nginx.h 修改第13-14行，如下示例
#define NGINX_VERSION "1.68.9"
#define NGINX_VER "wanginx/" NGINX_VERSION
如果server_tokens off，修改 src/http/ngx_http_header_filter_module.c
第49行，如下示例：
static char ngx_http_server_string[] = "Server: nginx" CRLF;
把其中的nginx改为自己想要的文字即可,如：wanginx


```

## 2.5 搭建虚拟主机

```
创建网站主页目录
[root@node2 ~]# mkdir -pv /data/{site1,site2}
[root@node2 ~]# echo /data/site1/index.html > /data/site1/index.html
[root@node2 ~]# echo /data/site2/index.html > /data/site2/index.html
配置虚拟主机
[root@node2 ~]# cat /etc/nginx/conf.d/test.conf 
charset utf-8;
server_tokens off;
server {
   server_name www.magedu.net;
   root /data/site1/;

}
server {
   server_name www.magedu.tech;
   root /data/site2/;

}
配置hosts文件解析
[root@node2 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.112  www.magedu.net  www.magedu.tech


[root@node2 ~]# curl www.magedu.net
/data/site1/index.html
[root@node2 ~]# curl www.magedu.tech
/data/site2/index.html

```

## 2.6 对主机名字匹配

```
server_name name ...;
 虚拟主机的主机名称后可跟多个由空白字符分隔的字符串
 支持*通配任意长度的任意字符
server_name *.magedu.com www.magedu.*
 支持~起始的字符做正则表达式模式匹配，性能原因慎用
server_name ~^www\d+\.magedu\.com$
说明： \d 表示 [0-9]
 匹配优先级机制从高到低
(1) 首先是字符串精确匹配 如：www.magedu.com
(2) 左侧*通配符 如：*.magedu.com
(3) 右侧*通配符 如：www.magedu.*
(4) 正则表达式 如： ~^.*\.magedu\.com$
(5) default_server
```

## 2.7 location 匹配规则

```
[root@node2 ~]# mkdir -p /opt/testdir/test/
[root@node2 ~]# echo welcome to test > /opt/testdir/test/index.html
[root@node2 ~]# curl www.magedu.net/test/index.html
welcome to test
[root@node2 ~]# cat /etc/nginx/conf.d/test.conf 
charset utf-8;
server_tokens off;
server {
   server_name www.magedu.net;
   root /data/site1/;
   location /test {
      root /opt/testdir;
   }   


}
server {
   server_name www.magedu.tech;
   root /data/site2/;

}

```

```
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}

The “/” request will match configuration A, the “/index.html” request will match configuration B, the “/documents/document.html” request will match configuration C, the “/images/1.gif” request will match configuration D, and the “/documents/1.jpg” request will match configuration E.
```

## 2.8 location 生产案例

```
生产案例：静态资源配置
location ^~ /static/ {
root /data/nginx/static;
}
# 或者
location ~* \.(gif|jpg|jpeg|png|bmp|tiff|tif|css|js|ico)$ {
root /data/nginx/static;
}
```

## 2.9 alias 别名

```
[root@node2 testdir]# cat /etc/nginx/conf.d/test.conf 
charset utf-8;
server_tokens off;
server {
   server_name www.magedu.net;
   location /about {
   	root /opt/testdir;
   }
}
[root@node2 testdir]# mkdir /opt/testdir/about
[root@node2 testdir]# echo web_about > /opt/testdir/about/index.html

[root@node2 testdir]# echo web_testdir > /opt/testdir/index.html 
[root@node2 testdir]# cat /etc/nginx/conf.d/test.conf 
charset utf-8;
server_tokens off;
server {
   server_name www.magedu.net;
   location /about {
   	alias /opt/testdir;
   }
}
[root@node2 testdir]# curl www.magedu.net/about/
web_testdir

```

## 3.1 错误日志

```
[root@node2 testdir]# cat /etc/nginx/conf.d/test.conf 
charset utf-8;
server_tokens off;
server {
   server_name www.magedu.net;
   root /opt/site1;
   error_page 404 /404.html;
   location = /404.html {
   }
   location /about {
   	root /opt/testdir;
   }
}

[root@node2 testdir]# mkdir -p /opt/site1
[root@node2 testdir]# echo web_error > /opt/site1/404.html
[root@node2 testdir]# nginx -s stop
[root@node2 testdir]# nginx
[root@node2 testdir]# curl www.magedu.net/about/sss
web_error

```

## 3.2 选择性的URI

```
[root@node2 ~]# mkdir /data/images/
[root@node2 ~]# cp /usr/share/pixmaps/faces/sunflower.jpg /data/images/a.jpg
[root@node2 ~]# cp /usr/share/pixmaps/faces/sky.jpg /data/images/b.jpg
[root@node2 ~]# cp /usr/share/pixmaps/faces/flower.jpg /data/images/default.jpg

配置host文件
C:\Windows\System32\drivers\etc\hosts

配置nginx

   location /images {
      alias /data/images;
      try_files $uri /images/default.jpg;
   }

浏览器访问：
http://www.magedu.net/images/a.jpg
http://www.magedu.net/images/b.jpg
http://www.magedu.net/images/c.jpg
```

## 3.3 连接时长

```
keepalive_timeout  65;
[root@node2 html]# telnet 192.168.0.112 80
Trying 192.168.0.112...
Connected to 192.168.0.112.
Escape character is '^]'.
GET / HTTP/1.1
host: 192.168.0.112

HTTP/1.1 200 OK
Server: nginx
Date: Tue, 16 Feb 2021 13:52:51 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 18
Last-Modified: Tue, 16 Feb 2021 01:09:14 GMT
Connection: keep-alive
ETag: "602b1b3a-12"
Accept-Ranges: bytes

welcome to magedu

```

## 3.4 允许请求的次数

```
keepalive_requests 2;

[root@node2 html]# telnet 192.168.0.112 80
Trying 192.168.0.112...
Connected to 192.168.0.112.
Escape character is '^]'.
GET / HTTP/1.1
host: 192.168.0.112

HTTP/1.1 200 OK
Server: nginx
Date: Tue, 16 Feb 2021 13:59:12 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 18
Last-Modified: Tue, 16 Feb 2021 01:09:14 GMT
Connection: keep-alive
ETag: "602b1b3a-12"
Accept-Ranges: bytes

welcome to magedu

GET / HTTP/1.1
host: 192.168.0.112

HTTP/1.1 200 OK
Server: nginx
Date: Tue, 16 Feb 2021 13:59:15 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 18
Last-Modified: Tue, 16 Feb 2021 01:09:14 GMT
Connection: close
ETag: "602b1b3a-12"
Accept-Ranges: bytes

welcome to magedu
Connection closed by foreign host.

```

## 3.5 对客户端限制速率

```
[root@node2 ~]# cd /data/site1/
[root@node2 site1]# dd if=/dev/zero of=test.img bs=1M count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB) copied, 0.355465 s, 295 MB/s

[root@node2 site1]# cat /etc/nginx/conf.d/test.conf
charset utf-8;
server_tokens off;
server {
   server_name www.magedu.net;
   root /data/site1;
   limit_rate 10k;
   error_page 404 /404.html;

   location /images {
      alias /data/images;
      try_files $uri /images/default.jpg;
   }

   location = /404.html {
   }
   location /about {
   	root /opt/testdir;
   }
}

[root@node2 site1]# wget http://www.magedu.net/test.img

```

## 3.6 限制客户端的请求方法

```
[root@node2 site1]# curl -X OPTIONS -I www.magedu.net
HTTP/1.1 403 Forbidden
Server: nginx
Date: Wed, 17 Feb 2021 03:06:03 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 146
Connection: keep-alive

[root@node2 site1]# cat /etc/nginx/conf.d/test.conf
charset utf-8;
server_tokens off;
server {
   server_name www.magedu.net;
   root /data/site1;
   limit_rate 10k;
   error_page 404 /404.html;
   location / {
     limit_except GET {
        #allow 192.168.0.112;
        deny all;
     }
   }

   location /images {
      alias /data/images;
      try_files $uri /images/default.jpg;
   }

   location = /404.html {
   }
   location /about {
   	root /opt/testdir;
   }
}

```

## 3.7 文件优化设置

```
文件操作优化的配置
aio on | off | threads[=pool];
是否启用aio功能，默认off
directio size | off;
当文件大于等于给定大小时，同步（直接）写磁盘，而非写缓存，默认off
示例：
location /video/ {
sendfile on;
aio on;
directio 8m;
}
ngx_http_core_module
```

## 3.8 ngx_http_access_module模块

```
location /about {
root /data/nginx/html/pc;
index index.html;
deny 192.168.1.1;
allow 192.168.1.0/24;
allow 10.1.1.0/16;
allow 2001:0db8::/32;
deny all; #先允许小部分，再拒绝大部分
}
```

## 3.9 ngx_http_auth_basic_module模块

```
[root@node2 nginx]# yum -y install httpd-tools
[root@node2 nginx]# htpasswd -c /etc/nginx/conf.d/.nginx_passwd alice
New password: 
Re-type new password: 
Adding password for user alice
[root@node2 nginx]# htpasswd -b /etc/nginx/conf.d/.nginx_passwd bob shaoxh
Adding password for user bob

[root@node2 nginx]# vim /etc/nginx/conf.d/test.conf  
 location /admin {
       root /data/;
       #deny 192.168.0.104   
       auth_basic "Admin Area";
       auth_basic_user_file /etc/nginx/conf.d/.nginx_passwd;
   }
[root@node2 nginx]# mkdir /data/admin
[root@node2 nginx]# echo shaoxh > /data/admin/index.htm

浏览器访问：
http://www.magedu.net/admin

[root@node2 nginx]# curl -u bob:shaoxh http://www.magedu.net/admin/
shaoxh


```

## 4.1  ngx_http_stub_status_module模块

```
location /nginx_status {
stub_status;
allow 127.0.0.1;
allow 192.168.0.112/24;
deny all;
}
```

## 4.2 nginx 第三方模块

```
[root@node2 src]# git clone https://github.com/openresty/echo-nginx-module.git
[root@node2 src]# yum -y install perl-devel perl-ExtUtils-Embed

 ./configure \
--prefix=/apps/nginx \
--user=nginx --group=nginx \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-http_perl_module \
--with-pcre \
--with-stream \
--with-stream_ssl_module \
--with-stream_realip_module \
--add-module=/usr/local/src/echo-nginx-module
 make && make install
 
  location /echo {
            echo hello world;
            default_type text/html;
            echo  "hello world,main-->";
        }

```

