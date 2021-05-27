# FastDFS

FastDFS是一个开源的轻量级[分布式文件系统](https://baike.baidu.com/item/分布式文件系统/1250388)，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和[负载均衡](https://baike.baidu.com/item/负载均衡/932451)的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。

FastDFS为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务。



## FastDFS：

​		Tracker：调度器，负责维持集群的信息，例如各group及其内部的storage node, 这些信息也是storage node报告所生成；每个storage node 会周期性向                        tracker 发心跳信息；   

​		Storage server: 以group为单位进行组织，任何一个storage Server都应该属于某个group，一个group因该包含多个storage server；在同一个group内部，各storage server 的数据互相冗余;

文件访问操作：upload,download,append,delete;

FID：

​		group/M00/00/00/FILE_ID

### Upload File :

 1. 由客户端发起上传连接请求；

 2. 由tracker 查找可用的storage server；

 3. 找到可用的storage server 后，将（ip：port） 返回给client；

 4. 上传文件 （文件的属性信息和文件内容）；

 5. 生成文件的fid，将client提交的内容写入选定位置；

 6. 返回fid；

 7. 同步存储文件信息至同组中的其它节点；

    

tracker 如何挑选组：

 1. rr

 2. 指定组

 3.  基于可用空间进行均衡

​    

如何在组中挑选storage server；

 1. rr

 2. 以ip为次序，找第一个

 3. 以优先级为序，找第一个

    

如何选择磁盘：

1. rr

2. 剩余可用空间大者优先

   

生成FID：

​	由源头storage server ip 、创建时的时间戳、大小、文件的校验码和一个随机数进行hash计算后生成；

文件同步：

 		每个storage server在文件存储完成后，会将其信息存于binlog，binlog 不包含数据，仅包含文件名等元数据信息；binlog可用于同步；



### Download File： 

​     客户端上传文件完成后，会收到storage server 返回的FID， 而后再次用到时，client会根据此文件发出请求；

		1. client 向tracker 发请求；
		2. tracker根据文件名定位到group，并返回此group内的某一个stroage server的信息（ip:port) 给client；
		3. client向得到的ip:port发请求
		4. storage server查找文件，并返回内容给client。



rpmfind.net

pkgs.org

rpm.pbone.net



## 源码编译安装fastdfs：

### 环境准备 

```
node1 192.168.0.169
node2 192.168.0.81
node3 192.168.0.100

yum -y install gcc automake autoconf libtool make
yum -y groupinstall "Development Tools" "Server Platform Development"

```

### node1 安装包

```
[root@node1 ~]# ll
total 688
-rw-------. 1 root root   1241 May 10 19:54 anaconda-ks.cfg
-rw-r--r--. 1 root root 338992 May 12 20:10 fastdfs-5.0.8-1.el7.centos.src.rpm
-rw-r--r--. 1 root root 335888 May 12 20:10 fastdfs-5.0.8.tar.gz
-rw-r--r--. 1 root root  17827 May 12 20:10 fastdfs-nginx-module-1.18.tar.gz

```

### 制作rpm

```
[root@node1 ~]# rpm -ivh fastdfs-5.0.8-1.el7.centos.src.rpm 
Updating / installing...
   1:fastdfs-5.0.8-1.el7.centos       ################################# [100%]
[root@node1 ~]# ls
anaconda-ks.cfg  fastdfs-5.0.8-1.el7.centos.src.rpm  fastdfs-5.0.8.tar.gz  fastdfs-nginx-module-1.18.tar.gz  rpmbuild
[root@node1 ~]# 

需要先安装libfastcommon
[root@node1 ~]# rpm -ivh libfastcommon-1.0.28-1.el7.centos.src.rpm 
Updating / installing...
   1:libfastcommon-1.0.28-1.el7.centos################################# [100%]
[root@node1 ~]# cd rpmbuild/SPECS/
[root@node1 SPECS]# ls
fastdfs.spec  libfastcommon.spec
[root@node1 SPECS]# rpmbuild -bb libfastcommon.spec 

查看生成的包
[root@node1 SPECS]# cd ..
[root@node1 rpmbuild]# ls
BUILD  BUILDROOT  RPMS  SOURCES  SPECS  SRPMS
[root@node1 rpmbuild]# ls RPMS/x86_64/
libfastcommon-1.0.28-1.el7.centos.x86_64.rpm            libfastcommon-devel-1.0.28-1.el7.centos.x86_64.rpm
libfastcommon-debuginfo-1.0.28-1.el7.centos.x86_64.rpm

安装libfastcommon
[root@node1 x86_64]# yum -y install libfastcommon-1.0.28-1.el7.centos.x86_64.rpm libfastcommon-devel-1.0.28-1.el7.centos.x86_64.rpm 

编译fastdfs
[root@node1 SPECS]# rpmbuild -bb fastdfs.spec 
出现错误需要修改 文件内容
%install
.......
#rm -f %{buildroot}/etc/fdfs/storage_ids.conf.sample
#%find_lang %{name}

[root@node1 SPECS]# rpmbuild -bb fastdfs.spec 

拷贝rpm包到指定的位置：
 mkdir /root/fdfs
 cp fastdfs-5.0.8-1.el7.centos.x86_64.rpm fastdfs-server-5.0.8-1.el7.centos.x86_64.rpm  fastdfs-tool-5.0.8-1.el7.centos.x86_64.rpm libfdfsclient-* /root/fdfs/   
 cd /root/fdfs/
 
 [root@node1 fdfs]# ll
total 360
-rw-r--r--. 1 root root   1816 May 13 11:03 fastdfs-5.0.8-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root 168780 May 13 11:03 fastdfs-server-5.0.8-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root 132200 May 13 11:03 fastdfs-tool-5.0.8-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root  36016 May 13 11:03 libfdfsclient-5.0.8-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root  18152 May 13 11:03 libfdfsclient-devel-5.0.8-1.el7.centos.x86_64.rpm
[root@node1 fdfs]# yum -y install *.rpm


```

### 生成的配置文件

```
[root@node1 fdfs]# rpm -ql fastdfs-server
/etc/fdfs/storage.conf.sample
/etc/fdfs/tracker.conf.sample
/etc/init.d/fdfs_storaged
/etc/init.d/fdfs_trackerd
/usr/bin/fdfs_storaged
/usr/bin/fdfs_trackerd
/usr/bin/restart.sh
/usr/bin/stop.sh
```

### 修改配置文件

```
root@node1 fdfs]# cd /etc/fdfs/
[root@node1 fdfs]# cp tracker.conf.sample tracker.conf
[root@node1 fdfs]# vim tracker.conf
base_path=/data/fastdfs
```

### 修改存储路径：

```
[root@node1 fdfs]# mkdir -pv /data/fastdfs
mkdir: created directory ‘/data’
mkdir: created directory ‘/data/fastdfs’
```

### 启动服务器

```
[root@node1 fdfs]# /etc/init.d/fdfs_trackerd start
Starting fdfs_trackerd (via systemctl):                    [  OK  ]
[root@node1 fdfs]# ss -tnl
*:22122      
[root@node1 fdfs]# tree /data/fastdfs/
/data/fastdfs/
├── data
│   ├── fdfs_trackerd.pid
│   └── storage_changelog.dat
└── logs
    └── trackerd.log

```

### 拷贝目录到node2

```
[root@node1 ~]# scp -r fdfs/ 192.168.0.81:/root
root@node1 ~]# cd rpmbuild/RPMS/x86_64/
[root@node1 x86_64]# ls
fastdfs-5.0.8-1.el7.centos.x86_64.rpm            libfastcommon-debuginfo-1.0.28-1.el7.centos.x86_64.rpm
fastdfs-debuginfo-5.0.8-1.el7.centos.x86_64.rpm  libfastcommon-devel-1.0.28-1.el7.centos.x86_64.rpm
fastdfs-server-5.0.8-1.el7.centos.x86_64.rpm     libfdfsclient-5.0.8-1.el7.centos.x86_64.rpm
fastdfs-tool-5.0.8-1.el7.centos.x86_64.rpm       libfdfsclient-devel-5.0.8-1.el7.centos.x86_64.rpm
libfastcommon-1.0.28-1.el7.centos.x86_64.rpm
[root@node1 x86_64]# scp libfastcommon-1.0.28-1.el7.centos.x86_64.rpm libfastcommon-devel-1.0.28-1.el7.centos.x86_64.rpm 192.168.0.81:/root/fdfs
root@192.168.0.81's password: 
libfastcommon-1.0.28-1.el7.centos.x86_64.rpm                                                                   100%   86KB  10.1MB/s   00:00    
libfastcommon-devel-1.0.28-1.el7.centos.x86_64.rpm                                                             100%   31KB  17.0MB/s   00:00  
```

## node2 安装fastdfs

```
[root@node2 ~]# cd fdfs/
[root@node2 fdfs]# ls
fastdfs-5.0.8-1.el7.centos.x86_64.rpm         libfastcommon-devel-1.0.28-1.el7.centos.x86_64.rpm
fastdfs-server-5.0.8-1.el7.centos.x86_64.rpm  libfdfsclient-5.0.8-1.el7.centos.x86_64.rpm
fastdfs-tool-5.0.8-1.el7.centos.x86_64.rpm    libfdfsclient-devel-5.0.8-1.el7.centos.x86_64.rpm
libfastcommon-1.0.28-1.el7.centos.x86_64.rpm
[root@node2 fdfs]# yum -y install *.rpm  
```

### 修改配置文件

```
[root@node2 fdfs]# cd /etc/fdfs/
[root@node2 fdfs]# ls
client.conf.sample  storage.conf.sample  tracker.conf.sample
[root@node2 fdfs]# cp storage.conf.sample storage.conf
[root@node2 fdfs]# vim storage.conf  
修改如下几行
base_path=/data/fastdfs
store_path_count=2
store_path0=/data/fastdfs/dev0
store_path1=/data/fastdfs/dev1
tracker_server=192.168.0.169:22122
```

### 创建所需要的目录

```
[root@node2 fdfs]# mkdir -pv /data/fastdfs/dev{0,1}
mkdir: created directory ‘/data’
mkdir: created directory ‘/data/fastdfs’
mkdir: created directory ‘/data/fastdfs/dev0’
mkdir: created directory ‘/data/fastdfs/dev1’
```

### 启动fdfs_storaged

```
[root@node2 fdfs]# /etc/init.d/fdfs_storaged start
```

## node3 安装fastdfs

```
和node2安装方式一致：
```

## 测试track_server

```
[root@node1 ~]# cd /etc/fdfs/
[root@node1 fdfs]# cp client.conf.sample client.conf
[root@node1 fdfs]# vim client.conf
[root@node1 fdfs]# vim client.conf
修改下面的值
base_path=/data/fastdfs
tracker_server=192.168.0.169:22122
```

### 上传

```
[root@node1 fdfs]# fdfs_test /etc/fdfs/client.conf upload /etc/fstab 

group_name=group1, remote_filename=M01/00/00/wKgAZGCcx_OABHuMAAAB0W3Dvac5748533

```

### 获取文件信息

```
[root@node1 fdfs]# fdfs_test /etc/fdfs/client.conf getmeta group1 M01/00/00/wKgAZGCcx_OABHuMAAAB0W3Dvac5748533
```

### 下载

```
[root@node1 fdfs]# cd /tmp
[root@node1 tmp]# fdfs_test /etc/fdfs/client.conf download group1 M01/00/00/wKgAZGCcx_OABHuMAAAB0W3Dvac5748533
[root@node1 tmp]# cat wKgAZGCcx_OABHuMAAAB0W3Dvac5748533 

#
# /etc/fstab
# Created by anaconda on Mon May 10 19:46:57 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=4ec88a3c-fc6f-4b47-8d0b-6afe9ca2a4b0 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0

```

```
[root@node1 ~]# tar xf fastdfs-5.0.8.tar.gz 
[root@node1 ~]# cd fastdfs-5.0.8
[root@node1 fastdfs-5.0.8]# ls
client  conf             fastdfs.spec  init.d   make.sh     README.md   stop.sh  test
common  COPYING-3_0.txt  HISTORY       INSTALL  php_client  restart.sh  storage  tracker
[root@node1 fastdfs-5.0.8]# cd conf/
[root@node1 conf]# ls
anti-steal.jpg  client.conf  http.conf  mime.types  storage.conf  storage_ids.conf  tracker.conf
[root@node1 conf]# cp mime.types http.conf anti-steal.jpg storage_ids.conf /etc/fdfs/
[root@node1 conf]# cd /etc/fdfs/
[root@node1 fdfs]# vim http.conf 
http.anti_steal.token_check_fail=/etc/fdfs/anti-steal.jpg
[root@node1 fdfs]# vim tracker.conf
include http.conf
[root@node1 fdfs]# /etc/init.d/fdfs_trackerd restart
[root@node1 fdfs]# scp http.conf mime.types anti-steal.jpg storage_ids.conf 192.168.0.81:/etc/fdfs/
[root@node1 fdfs]# scp http.conf mime.types anti-steal.jpg storage_ids.conf 192.168.0.100:/etc/fdfs/


```



## 编译安装nginx

```
[root@node2 ~]# ll *.tar.gz
-rw-r--r--. 1 root root  17827 May 13 15:14 fastdfs-nginx-module-1.18.tar.gz
-rw-r--r--. 1 root root 911509 May 13 15:13 nginx-1.10.3.tar.gz
[root@node2 ~]# tar xf fastdfs-nginx-module-1.18.tar.gz 
[root@node2 ~]# tar xf nginx-1.10.3.tar.gz 
[root@node2 ~]# cd nginx-1.10.3
[root@node2 nginx-1.10.3]# 
./configure \
--prefix=/usr \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/locak/nginx.lock \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/tmp/nginx/client/ \
--http-proxy-temp-path=/var/tmp/nginx/proxy/ \
--http-fastcgi-temp-path=/var/tmp/nginx/fcgi \
--http-uwsgi-temp-path=/var/tmp/nginx/uwsgi \
--http-scgi-temp-path=/var/tmp/nginx/scgi \
--with-pcre \
--with-debug \
--add-module=../fastdfs-nginx-module-1.18/src
make && make install

[root@node2 nginx-1.10.3]# yum -y install pcre-devel openssl-devel
[root@node2 nginx-1.10.3]# useradd -r nginx
[root@node2 nginx-1.10.3]# chown -R nginx.nginx /var/tmp/nginx/
[root@node2 nginx-1.10.3]# vim /etc/init.d/nginx
[root@node2 nginx-1.10.3]# chmod +x /etc/init.d/nginx
[root@node2 nginx-1.10.3]# chkconfig --add nginx


```

### 脚本文件内容

```
#!/bin/bash
#nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15 
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/usr/sbin/nginx"
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/etc/nginx/nginx.conf"
 
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
 
lockfile=/var/lock/subsys/nginx
 
make_dirs() {
   # make required directories
   user=`nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
 
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
 
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
 
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
 
force_reload() {
    restart
}
 
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
 
rh_status() {
    status $prog
}
 
rh_status_q() {
    rh_status >/dev/null 2>&1
}
 
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac

```

### nginx配置文件

```
[root@node2 nginx-1.10.3]# nginx -s stop
[root@node2 nginx-1.10.3]# cd /etc/nginx/
[root@node2 nginx]# ls
fastcgi.conf          fastcgi_params.default  mime.types          nginx.conf.default   uwsgi_params
fastcgi.conf.default  koi-utf                 mime.types.default  scgi_params          uwsgi_params.default
fastcgi_params        koi-win                 nginx.conf          scgi_params.default  win-utf

```

nginx.conf 添加

```
location /group1/M01 {
          root /data/fastdfs;
          ngx_fastdfs_module;
        }

```

### 上传图片

```
[root@node1 fdfs]# find /usr/share -iname "*.jpg"
/usr/share/kde4/apps/ksplash/Themes/CentOS7/2560x1600/background.jpg
[root@node1 fdfs]# fdfs_test /etc/fdfs/client.conf upload /usr/share/kde4/apps/ksplash/Themes/CentOS7/2560x1600/background.jpg
group_name=group1, remote_filename=M01/00/00/wKgAZGCc3DGAAje_AA6q2wjnW8s158.jpg
example file url: http://192.168.0.100/group1/M01/00/00/wKgAZGCc3DGAAje_AA6q2wjnW8s158.jpg

```

