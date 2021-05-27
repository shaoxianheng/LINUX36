# SElinux

## selinux 存放策略的文件

```
[root@node1 data]# ll /etc/selinux/targeted/
total 16
drwx------. 3 root root  206 Oct 26 11:07 active
-rw-r--r--. 1 root root 2623 Aug  6  2017 booleans.subs_dist
drwxr-xr-x. 4 root root 4096 Aug 27 21:06 contexts
drwxr-xr-x. 2 root root    6 Aug  6  2017 logins
drwxr-xr-x. 3 root root   20 Aug 27 21:06 modules
drwxr-xr-x. 2 root root   23 Oct 26 11:07 policy
-rw-------. 1 root root    0 Aug  6  2017 semanage.read.LOCK
-rw-------. 1 root root    0 Aug  6  2017 semanage.trans.LOCK
-rw-r--r--. 1 root root  607 Aug  6  2017 setrans.conf
-rw-r--r--. 1 root root  106 Oct 26 11:07 seusers
[root@node1 data]# ll /etc/selinux/targeted/policy/
total 3644
-rw-r--r--. 1 root root 3730486 Oct 26 11:07 policy.30

```

## 配置文件

```
[root@node1 data]# cat /etc/selinux/config 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 


[root@node1 data]# 

```

selinux 关闭状态下，文件属性最后面没有点

```
[root@node1 data]# getenforce 
Disabled
[root@node1 data]# 
[root@node1 data]# 
[root@node1 data]# touch file2
[root@node1 data]# ll
total 32700
-rwxr-xr-x. 1 root root       54 Oct 26 08:48 bak.sh
-rw-r--r--. 1 root root    80698 Oct 26 09:57 boot.xml
-rw-r--r--. 1 root root 33392640 Oct 26 09:10 etc.tar
-rw-r--r--  1 root root        0 Oct 26 11:43 file2
-rw-r--r--. 1 root root       41 Oct 25 19:50 txt
[root@node1 data]# 

```

## 下面是有selinux策略的和没有策略的

```
[root@node1 data]# ll -Z file2 
-rw-r--r--. root root system_u:object_r:default_t:s0   file2
[root@node1 data]# 
[root@node1 data]# 
[root@node1 data]# ll -Z txt 
-rw-r--r--. root root unconfined_u:object_r:default_t:s0 txt
[root@node1 data]# ll

```

```
[root@node1 data]# setenforce 
usage:  setenforce [ Enforcing | Permissive | 1 | 0 ]
[root@node1 data]# getenforce 
Disabled
[root@node1 data]# 
[root@node1 data]# 
[root@node1 data]# touch file3
[root@node1 data]# ll 
total 32700
-rwxr-xr-x. 1 root root       54 Oct 26 08:48 bak.sh
-rw-r--r--. 1 root root    80698 Oct 26 09:57 boot.xml
-rw-r--r--. 1 root root 33392640 Oct 26 09:10 etc.tar
-rw-r--r--. 1 root root        0 Oct 26 11:43 file2
-rw-r--r--  1 root root        0 Oct 26 11:49 file3
-rw-r--r--. 1 root root       41 Oct 25 19:50 txt
[root@node1 data]# ll -Z file3 
-rw-r--r-- root root ?                                file3
[root@node1 data]# 

```

## httpd页面访问

```
[root@node1 ~]# yum -y install httpd
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
Package httpd-2.4.6-93.el7.centos.x86_64 already installed and latest version
Nothing to do
[root@node1 ~]# systemctl start httpd
[root@node1 ~]# iptables -F
[root@node1 ~]# echo ggg > /var/www/html/index.html 
[root@node1 ~]# ll /var/www/html/index.html -Z
-rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
现在状态能访问


[root@node1 ~]# cp /var/www/html/index.html /root
[root@node1 ~]# mv /root/index.html /var/www/html/index.html 
mv: overwrite ‘/var/www/html/index.html’? y
[root@node1 ~]# ll /var/www/html/index.html
-rw-r--r--. 1 root root 4 Oct 26 12:26 /var/www/html/index.html
[root@node1 ~]# ll /var/www/html/index.html -Z
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 /var/www/html/index.html
[root@node1 ~]# 
现在状态不能访问

[root@node1 ~]# rm -f /var/www/html/index.html 
[root@node1 ~]# echo shaoxh > /var/www/html/index.html 
[root@node1 ~]# ll -Z /var/www/html/index.html 
-rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
[root@node1 ~]# 
现在状态能访问

```

## 恢复标签信息

```
[root@node1 ~]# cp /var/www/html/index.html /root/index.html
[root@node1 ~]# mv /root/index.html /var/www/html/index.html 
mv: overwrite ‘/var/www/html/index.html’? y
[root@node1 ~]# ll -Z /var/www/html/index.html 
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 /var/www/html/index.html
[root@node1 ~]# restorecon /var/www/html/index.html 
[root@node1 ~]# ll -Z /var/www/html/index.html 
-rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
[root@node1 ~]# 

```

## getsebool命令

```
getsebool命令是用来查询SElinux策略内各项规则的布尔值。SELinux的策略与规则管理相关命令：seinfo命令、sesearch命令、getsebool命令、setsebool命令、semanage命令。下面让我们详细讲解一下getsebool命令的使用方法。
[root@node1 ~]# getsebool -a |grep ftp
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off
[root@node1 ~]# 

```

## selinux 状态

```
[root@node1 ~]# sestatus 
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
[root@node1 ~]# 

```































