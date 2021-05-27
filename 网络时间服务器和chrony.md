## 网络时间服务器和chrony

## ntp

```
[root@node1 ~]# rpm -qi ntp
Name        : ntp
Version     : 4.2.6p5
Release     : 29.el7.centos.2
Architecture: x86_64
Install Date: Sun 01 Nov 2020 05:46:15 PM CST
Group       : System Environment/Daemons

```



## chrony

```
[root@node1 ~]# rpm -qi chrony
Name        : chrony
Version     : 3.1

```

## date

```
[root@node1 ~]# date -s "1 year"
Mon Nov  1 17:56:07 CST 2021
[root@node1 ~]# clock
Sun 01 Nov 2020 05:56:14 PM CST  -0.507852 seconds
[root@node1 ~]# poweroff

```

![image-20201101175821630](C:\Users\localhost\AppData\Roaming\Typora\typora-user-images\image-20201101175821630.png)

## 系统自动同步硬件时间

![image-20201101180024870](C:\Users\localhost\AppData\Roaming\Typora\typora-user-images\image-20201101180024870.png)

修改硬件时间关机后时间

![image-20201101180303894](C:\Users\localhost\AppData\Roaming\Typora\typora-user-images\image-20201101180303894.png)

![image-20201101180411688](C:\Users\localhost\AppData\Roaming\Typora\typora-user-images\image-20201101180411688.png)

## ntpdate同步时间

```
[root@node1 ~]# ntpdate ntp.sjtu.edu.cn     上海交通大学
 1 Nov 18:40:59 ntpdate[14314]: adjust time server 203.107.6.88 offset -0.001709 sec
```

## ntp服务器

```
[root@node1 ~]# yum -y install ntp
[root@node1 ~]# systemctl enable ntpd
[root@node1 ~]# systemctl start ntpd
[root@node1 ~]# grep ibu /etc/ntp.conf
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

如果想让别人同步你的时间，注释
#restrict default nomodify notrap nopeer noquery


```

## ntpq 查看同步的信息

```
[root@node1 ~]# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 203.107.6.88    100.107.25.114   2 u    -   64    1   15.303   65.348   0.093
 ntp8.flashdance .INIT.          16 u    -   64    0    0.000    0.000   0.000
 time.cloudflare .INIT.          16 u    -   64    0    0.000    0.000   0.000
[root@node1 ~]# 


```

## 客户端配置

```
[root@node2 ~]# grep iburst /etc/ntp.conf 
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server 192.168.1.128 iburst


[root@node2 ~]# ntpq -p       *号代表已经同步
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*192.168.1.128   116.203.151.74   3 u   41   64    1    0.431   -1.412   1.388

```

## chrony配置

```
网路同步时间服务器选择   上海交大
server ntp.sjtu.edu.cn 
```

```
root@node3 ~]# yum -y install chronyd

[root@node3 ~]# grep iburst /etc/chrony.conf 
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server ntp.sjtu.edu.cn iburst

allow 192.168.1.0/24   打开此网段，才能使他人与你同步

local stratum 10    如果你的网路断了，本机也可以当时间服务器让他人同步（10可以嵌套10层当服务器）


[root@node3 ~]# systemctl start chronyd
[root@node3 ~]# systemctl enable chronyd



```

## 验证node3服务器上的chrony服务是否能够同步

```
[root@node4 ~]# ntpdate 192.168.1.130
 1 Nov 08:25:40 ntpdate[10423]: adjust time server 192.168.1.130 offset 0.012968 sec
[root@node4 ~]# 

```

## chronyc 查看信息

```
[root@node4 ~]# chronyc sources -v
210 Number of sources = 4

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? ntp6.flashdance.cx            0   6     0     -     +0ns[   +0ns] +/-    0ns
^- a.chl.la                      2   6    17     2    -10ms[  -10ms] +/-  113ms
^* 203.107.6.88                  2   6    17     4   +880us[+2940us] +/-   15ms
^- electrode.felixc.at           3   6    13     2  +9929us[+9929us] +/-  154ms
[root@node4 ~]# 

```

## chronyc 命令  -n选项不解析主机名

```
测试那些主机可以来同步时间
[root@localhost ~]# chronyc 
chrony version 3.4
Copyright (C) 1997-2003, 2007, 2009-2018 Richard P. Curnow and others
chrony comes with ABSOLUTELY NO WARRANTY.  This is free software, and
you are welcome to redistribute it under certain conditions.  See the
GNU General Public License version 2 for details.

chronyc> accheck 192.168.1.131
208 Access allowed
chronyc> accheck 192.168.10.131
209 Access denied
chronyc> 

```

总结：nft服务器设置2项；客户端设置1项；

​            chrony服务器设置3项   客户端设置1项；

集群服务初始

1.设置防火墙

2.设置色Linux

3.设置时间服务器

