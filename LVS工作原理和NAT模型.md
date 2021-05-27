# LVS工作原理和NAT模型 

```
查看是否支持IPVS

[root@node1 ~]# grep -i ipvs -A 10 /boot/config-3.10.0-693.el7.x86_64 
CONFIG_NETFILTER_XT_MATCH_IPVS=m
CONFIG_NETFILTER_XT_MATCH_LENGTH=m
CONFIG_NETFILTER_XT_MATCH_LIMIT=m
CONFIG_NETFILTER_XT_MATCH_MAC=m

```

##  NAT 模型的验证

```
准备环境
node1                 node2                node3                node4             node5  
客户端                  drect                rs1                 rs2               mysql
桥接                   桥接和nat              nat                 nat               nat
192.168.0.109         192.168.0.111         192.168.137.131   
                      192.168.137.129                        192.168.137.130
                                           gw
                                           192.168.137.129   192.168.137.129
                                          
 
rs1和rs2安装httpd服务
[root@node4 ~]# yum -y install httpd
[root@node4 ~]# echo rs2 > /var/www/html/index.html
[root@node4 ~]# systemctl start httpd

验证调度服务器是否能够访问
[root@node2 ~]# curl 192.168.137.130
rs2
[root@node2 ~]# curl 192.168.137.131
rs1

打开转发功能
[root@node2 ~]# cat /etc/sysctl.conf
net.ipv4.ip_forward=1

安装ipvsadm
[root@node2 ~]# yum -y install ipvsadm

添加规则
[root@node2 ~]# ipvsadm -A -t 192.168.0.111:80 -s rr
[root@node2 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.111:80 rr
[root@node2 ~]# ipvsadm -a -t 192.168.0.111:80 -r 192.168.137.131 -m
[root@node2 ~]# ipvsadm -a -t 192.168.0.111:80 -r 192.168.137.130 -m
[root@node2 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.111:80 rr
  -> 192.168.137.130:80           Masq    1      0          0         
  -> 192.168.137.131:80           Masq    1      0          0        

[root@node2 ~]# ipvsadm -Ln --stats
[root@node2 ~]# ipvsadm -Ln --rate

客户端验证访问
[root@node1 ~]# while true; do curl 192.168.0.111;sleep 0.5;done
rs2
rs1
rs2
rs1
rs2

修改调度算法
[root@node2 ~]# ipvsadm -E -t 192.168.0.111:80 -s wrr
[root@node2 ~]# 

修改权重
[root@node2 ~]# ipvsadm -e -t 192.168.0.111:80 -r 192.168.137.130 -m -w 6
[root@node2 ~]# ipvsadm -e -t 192.168.0.111:80 -r 192.168.137.131 -m -w 2

验证：
[root@node1 ~]# while true; do curl 192.168.0.111;sleep 0.5;done
rs2
rs1
rs2
rs2
rs2
rs1
rs2
rs2
rs2

修改rs1和rs2 服务端口号
[root@node3 ~]# sed -i 's/^Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf 
[root@node3 ~]# systemctl restart httpd

[root@node4 ~]# sed -i 's/^Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf 
[root@node4 ~]# systemctl restart httpd

删除80的端口
[root@node2 ~]# ipvsadm -d -t 192.168.0.111:80 -r 192.168.137.130
[root@node2 ~]# ipvsadm -d -t 192.168.0.111:80 -r 192.168.137.131

添加8080的规则
[root@node2 ~]# ipvsadm -a -t 192.168.0.111:80 -r 192.168.137.131:8080 -m -w 6
[root@node2 ~]# ipvsadm -a -t 192.168.0.111:80 -r 192.168.137.130:8080 -m -w 6

验证：
[root@node1 ~]# while true; do curl 192.168.0.111;sleep 0.5;done
rs2
rs1
rs2
rs1
rs2

修改调度算法为sh
[root@node2 ~]# ipvsadm -E -t 192.168.0.111:80 -s sh

验证
[root@node1 ~]# while true; do curl 192.168.0.111;sleep 0.5;done
rs1
rs1
rs1
rs1

调度算法dh


```

### 搭建wordpress

```
[root@node3 ~]# yum -y install php-fpm  php-mysql
[root@node4 ~]# yum -y install php-fpm  php-mysql
[root@node5 ~]# yum -y install mariadb-server
[root@node5 ~]# systemctl start mariadb
[root@node5 ~]# mysql -e "create database wordpress"
[root@node5 ~]# mysql -e "grant all on wordpress.* to wordpress@'192.168.137.%' identified by 'shaoxh'"

[root@node5 ~]# vi /etc/my.cnf
skip_name_resolve=ON
跳过名称解析
[root@node5 ~]# systemctl restart mariadb
[root@node5 ~]# mysql -uwordpress -pshaoxh -h192.168.137.132

配置反向代理
[root@node3 yum.repos.d]# systemctl start php-fpm
[root@node3 yum.repos.d]# vim /etc/httpd/conf.d/fcgi.conf
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps
ProxyRequests Off
ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/var/www/html/$1

默认打开的程序类型
[root@node3 yum.repos.d]# vim /etc/httpd/conf/httpd.conf 
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
[root@node3 yum.repos.d]# systemctl restart httpd php-fpm

解压wordpress程序
[root@node3 html]# ls
index.html  wordpress-5.0.3-zh_CN.tar.gz
[root@node3 html]# 
[root@node3 html]# tar xf wordpress-5.0.3-zh_CN.tar.gz 
[root@node3 html]# cd wordpress/
[root@node3 html]# ls
index.html  wordpress-5.0.3-zh_CN.tar.gz
[root@node3 html]# 
[root@node3 html]# tar xf wordpress-5.0.3-zh_CN.tar.gz 
[root@node3 html]# cd wordpress/
[root@node3 wordpress]# vim wp-config.php

/** WordPress数据库的名称 */
define('DB_NAME', 'wordpress');

/** MySQL数据库用户名 */
define('DB_USER', 'wordpress');

/** MySQL数据库密码 */
define('DB_PASSWORD', 'shaoxh');

/** MySQL主机 */
define('DB_HOST', '192.168.137.132');


先测试一台主机131的wordpress，
[root@node2 ~]# ipvsadm -d -t 192.168.0.111:80 -r 192.168.137.130:8080
[root@node2 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.111:80 sh
  -> 192.168.137.131:8080         Masq    6      0          0      


浏览器打开
http://192.168.0.111/wordpress/

拷贝node3 wordpress目录到node4
 sed -i 's/^Listen 8080/Listen 80/' /etc/httpd/conf/httpd.conf 
 scp /etc/httpd/conf.d/fcgi.conf 192.168.137.130:/etc/httpd/conf.d/
 scp -r wordpress 192.168.137.130:/var/www/html/
 scp  /etc/httpd/conf/httpd.conf 192.168.137.130:/etc/httpd/conf
 systemctl restart httpd php-fpm

删除原来的ipvs规则
[root@node2 ~]# ipvsadm -d -t 192.168.0.111:80 -r 192.168.137.131:8080

[root@node2 ~]# ipvsadm -a -t 192.168.0.111:80 -r 192.168.137.131:80 -m
[root@node2 ~]# ipvsadm -a -t 192.168.0.111:80 -r 192.168.137.130:80 -m
[root@node2 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.111:80 rr
  -> 192.168.137.130:80           Masq    1      1          4         
  -> 192.168.137.131:80           Masq    1      0          5         

浏览器打开
http://192.168.0.111/wordpress/


保存启动ipvsadm
[root@node2 ~]# ipvsadm-save > /etc/sysconfig/ipvsadm
[root@node2 ~]# cat /etc/sysconfig/ipvsadm
-A -t 192.168.0.111:80 -s rr
-a -t 192.168.0.111:80 -r 192.168.137.130:80 -m -w 1
-a -t 192.168.0.111:80 -r 192.168.137.131:80 -m -w 1

[root@node2 ~]# systemctl start ipvsadm


```

## DR模型

```
node1             node2              node3            node4                  node5
桥接               桥接和nat            nat              nat                   nat
192.168.0.109     192.168.0.111      192.168.137.131   192.168.137.130       192.168.137.132 
                  192.168.137.129 
gw
192.168.0.111                                                                192.168.137.129
客户端              路由器             rs1                    rs2               lvs

VIP 192.168.137.100/24


node1配置指向node2的网关
AGTEWAY=192.168.0.111

node2 启用路由器的转发功能
[root@node2 ~]# cat /etc/sysctl.conf 
net.ipv4.ip_forward=1

node3 配置指向node2的网关
AGTEWAY=192.168.0.111

现在node1就可以和node3.node4 通信
[root@node1 ~]# curl 192.168.137.130/index.html
rs2
[root@node1 ~]# curl 192.168.137.131/index.html
rs1

配置lvs的网关
[root@node5 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33 
IPADDR=192.168.137.132
GATEWAY=192.168.137.129
NETMASK=255.255.255.0

lvs 配置vip地址
[root@node5 ~]# ip addr add 192.168.137.100/24 dev ens33


RS 配置VIP地址的脚本

[root@node3 ~]# cat lvs_dr_rs.sh
#!/bin/bash
#Author:wangxiaochun
#Date:2017-08-13
vip=192.168.137.100
mask='255.255.255.255'
dev=lo:1
rpm -q httpd &> /dev/null || yum -y install httpd &>/dev/null
#service httpd start &> /dev/null && echo "The httpd Server is Ready!"
#echo "<h1>`hostname`</h1>" > /var/www/html/index.html

case $1 in
start)
    echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
    ifconfig $dev $vip netmask $mask #broadcast $vip up
    #route add -host $vip dev $dev
    echo "The RS Server is Ready!"
    ;;
stop)
    ifconfig $dev down
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
    echo "The RS Server is Canceled!"
    ;;
*) 
    echo "Usage: $(basename $0) start|stop"
    exit 1
    ;;
esac

执行脚本
[root@node3 ~]# bash lvs_dr_rs.sh start
The RS Server is Ready!

拷贝脚本文件到rs2
[root@node3 ~]# scp lvs_dr_rs.sh 192.168.137.130:/root
执行脚本
[root@node4 ~]# bash lvs_dr_rs.sh start
The RS Server is Ready!


lvs安装ipvsadm
[root@node5 ~]# yum -y install ipvsadm

添加ipvs规则
[root@node5 ~]# ipvsadm -A -t 192.168.137.100:80 -s rr
[root@node5 ~]# ipvsadm -a -t 192.168.137.100:80 -r 192.168.137.131
[root@node5 ~]# ipvsadm -a -t 192.168.137.100:80 -r 192.168.137.130
[root@node5 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.137.100:80 rr
  -> 192.168.137.131:80           Route   1      0          0         
  -> 192.168.137.130:80           Route   1      0          0    
  
  
验证
[root@node1 ~]# while true;do curl http://192.168.137.100/index.html;sleep 0.5;done
rs2
rs1
rs2
rs1
rs2
rs1

lvs使用脚本配置
[root@node5 ~]# bash lvs_dr_vs.sh start
[root@node5 ~]# cat lvs_dr_vs.sh 
#!/bin/bash
#Author:wangxiaochun
#Date:2017-08-13
vip='192.168.137.100'
iface='lo:1'
mask='255.255.255.255'
port='80'
rs1='192.168.137.131'
rs2='192.168.137.130'
scheduler='wrr'
type='-g'
rpm -q ipvsadm &> /dev/null || yum -y install ipvsadm &> /dev/null

case $1 in
start)
    ifconfig $iface $vip netmask $mask #broadcast $vip up
    iptables -F
 
    ipvsadm -A -t ${vip}:${port} -s $scheduler
    ipvsadm -a -t ${vip}:${port} -r ${rs1} $type -w 1
    ipvsadm -a -t ${vip}:${port} -r ${rs2} $type -w 1
    echo "The VS Server is Ready!"
    ;;
stop)
    ipvsadm -C
    ifconfig $iface down
    echo "The VS Server is Canceled!"
    ;;
*)
    echo "Usage: $(basename $0) start|stop"
    exit 1
    ;;
esac

[root@node1 ~]# while true;do curl http://192.168.137.100/index.html;sleep 0.5;done
rs1
rs2
rs1
rs2
rs1
rs2
rs1
rs2




```

##  DR模型不在同一网段

```
node1             node2             node5              node3                          node4
cip               route              lvs                rs1                           rs2
192.168.0.109     192.168.0.111     192.168.137.132    192.168.137.131       192.168.137.130
                  192.168.137.129  
                  10.0.0.200/8
gw 
192.168.0.111                      192.168.137.129      192.168.137.129      192.168.137.129
vip  
10.0.0.100


node2添加10网段的ip地址
[root@node2 ~]# nmcli connection modify "Wired connection 1" +ipv4.addresses 10.0.0.200/8
[root@node2 ~]# nmcli connection up "Wired connection 1"

rs的脚本
[root@node3 ~]# cat lvs_dr_rs.sh 
#!/bin/bash
#Author:wangxiaochun
#Date:2017-08-13
vip=10.0.0.100
mask='255.255.255.255'
dev=lo:1
rpm -q httpd &> /dev/null || yum -y install httpd &>/dev/null
#service httpd start &> /dev/null && echo "The httpd Server is Ready!"
#echo "<h1>`hostname`</h1>" > /var/www/html/index.html

case $1 in
start)
    echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
    ifconfig $dev $vip netmask $mask #broadcast $vip up
    #route add -host $vip dev $dev
    echo "The RS Server is Ready!"
    ;;
stop)
    ifconfig $dev down
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
    echo "The RS Server is Canceled!"
    ;;
*) 
    echo "Usage: $(basename $0) start|stop"
    exit 1
    ;;
esac

lvs的脚本文件
[root@node5 ~]# cat lvs_dr_vs.sh 
#!/bin/bash
#Author:wangxiaochun
#Date:2017-08-13
vip='10.0.0.100'
iface='lo:1'
mask='255.255.255.255'
port='80'
rs1='192.168.137.131'
rs2='192.168.137.130'
scheduler='wrr'
type='-g'
rpm -q ipvsadm &> /dev/null || yum -y install ipvsadm &> /dev/null

case $1 in
start)
    ifconfig $iface $vip netmask $mask #broadcast $vip up
    iptables -F
 
    ipvsadm -A -t ${vip}:${port} -s $scheduler
    ipvsadm -a -t ${vip}:${port} -r ${rs1} $type -w 1
    ipvsadm -a -t ${vip}:${port} -r ${rs2} $type -w 1
    echo "The VS Server is Ready!"
    ;;
stop)
    ipvsadm -C
    ifconfig $iface down
    echo "The VS Server is Canceled!"
    ;;
*)
    echo "Usage: $(basename $0) start|stop"
    exit 1
    ;;
esac


验证:
[root@node1 ~]# while true;do curl http://10.0.0.100/index.html;sleep 0.5;done
rs2
rs1
rs2
rs1
rs2
rs1
rs2


```

### 使用443端口

```
node3和node4 都执行
[root@node3 ~]# yum -y install mod_ssl
[root@node3 ~]# systemctl restart httpd
[root@node3 ~]# ss -tnl

定义ipvs调度443的规则
[root@node5 ~]# ipvsadm -A -t 10.0.0.100:443 -s rr
[root@node5 ~]# ipvsadm -a -t 10.0.0.100:443 -r 192.168.137.130
[root@node5 ~]# ipvsadm -a -t 10.0.0.100:443 -r 192.168.137.131
[root@node5 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.100:80 wrr
  -> 192.168.137.130:80           Route   1      0          1         
  -> 192.168.137.131:80           Route   1      0          2         
TCP  10.0.0.100:443 rr
  -> 192.168.137.130:443          Route   1      0          0         
  -> 192.168.137.131:443          Route   1      0          0         

验证
[root@node1 ~]# while true;do curl -k https://10.0.0.100/index.html;sleep 0.5;done
rs1
rs2
rs1
rs2


ipvs打标签
[root@node5]# iptables -t mangle -A PREROUTING -d 10.0.0.100 -p tcp -m multiport --dports 80,443 -j MARK --set-mark 10
[root@node5]# ipvsadm -a -f 10 -r 192.168.137.131 -g
[root@node5]# ipvsadm -a -f 10 -r 192.168.137.130 -g
[root@node5 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
FWM  10 rr
  -> 192.168.137.130:0            Route   1      1          0         
  -> 192.168.137.131:0            Route   1      0          0         
[root@node5 ~]# 

验证：
[root@node1 ~]# !while
while true;do curl -k https://10.0.0.100/index.html;sleep 0.5;done
rs2
rs1
rs2
rs1
rs2
rs1

持久连接：
[root@node5 ~]# ipvsadm -E -f 10 -s rr -p
[root@node5 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
FWM  10 rr persistent 360
  -> 192.168.137.130:0            Route   1      2          0         
  -> 192.168.137.131:0            Route   1      0          0      
  
验证：
[root@node1 ~]# !while
while true;do curl -k https://10.0.0.100/index.html;sleep 0.5;done
rs1
rs1
rs1
rs1
rs1
rs1
rs1
rs1

```

### 安装ldirectord

```
[root@node5 ~]# yum -y install ldirectord-3.9.6-0rc1.1.1.x86_64.rpm 
[root@node5 ~]# rpm -ql ldirectord
[root@node5 ~]# cp /usr/share/doc/ldirectord-3.9.6/ldirectord.cf /etc/ha.d/

配置Sorry Server
[root@node5 ~]# yum -y install httpd
[root@node5 ~]# echo Sorry Server > /var/www/html/index.html
[root@node5 ~]# systemctl start httpd

先停止清楚lvs的规则，配置vip
[root@node5 ~]# bash lvs_dr_vs.sh stop
[root@node5 ~]# ifconfig lo:1 10.0.0.100 netmask 255.255.255.255

启动ldirectord服务
[root@node5 ~]# grep -vE "^$|#" /etc/ha.d/ldirectord.cf
checktimeout=3
checkinterval=1
autoreload=yes
logfile="/var/log/ldirectord.log"
quiescent=no
virtual=10.0.0.100:80
	real=192.168.137.131:80 gate
	real=192.168.137.130:80 gate
	fallback=127.0.0.1:80 gate
	service=http
	scheduler=rr
	protocol=tcp
	checktype=negotiate
	checkport=80
	request="index.html"
	
[root@node5 ha.d]# systemctl start ldirectord
[root@node5 ha.d]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.100:80 rr
  -> 192.168.137.130:80           Route   1      0          0         
  -> 192.168.137.131:80           Route   1      0          0     	

验证
[root@node1 ~]# while true;do curl http://10.0.0.100/index.html;sleep 0.5;done
rs1
rs2
rs1
rs2

停止131 的rs服务器
[root@node3 ~]# systemctl stop httpd

验证：
[root@node1 ~]# while true;do curl http://10.0.0.100/index.html;sleep 0.5;done
rs2
rs2
rs2
rs2
rs2
rs2

已经剔除了131 失败的服务
[root@node5 ha.d]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.100:80 rr
  -> 192.168.137.130:80           Route   1      0          20     

```

## 

