# Keepalived

## 配置keepalived

```
配置环境
client        route         lvs1    lvs2           rs1          rs2     
node1         node2          node3  node4        node5        node6
桥接          桥接和nat      nat      nat           nat           nat

0.6          0.7           137.17   137.27       137.37        137.47
             137.7     
gw          
0.7                        37.7     37.7         37.7          37.7

配置iptables selinux 关闭 基于key访问
安装keepalived服务
 
[root@node3 ~]# yum -y install keepalived
[root@node4 ~]# yum -y install keepalived

备份keepalived的配置文件
[root@node3 ~]# cp /etc/keepalived/keepalived.conf{,.bak}

配置keepalived之间的相互通信
[root@node3 ~]# ssh-copy-id -i .ssh/id_rsa.pub 192.168.137.27
[root@node4 ~]# ssh-copy-id 192.168.137.17



```

### 全局配置和VRRP协议的配置

```
[root@node3 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   notification_email {
	root@localhost
   }
   notification_email_from keepalived@localhost 
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_mcast_group4 224.100.100.100
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.137.100/24 dev ens33 label ens33:1
    }
}


[root@node4 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   notification_email {
	root@localhost
   }
   notification_email_from keepalived@localhost 
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_mcast_group4 224.100.100.100
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.137.100/24 dev ens33 label ens33:1
    }
}


```

### 测试VRRP广播

```
[root@node5 ~]# tcpdump -i eth0 -nn host 224.100.100.100

启动node4 backup的keepalived 服务查看情况
[root@node4 ~]# systemctl start keepalived
[root@node4 ~]# ip addr list
[root@node4 ~]# ip addr list|grep inet
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 192.168.137.27/24 brd 192.168.137.255 scope global ens33
    inet 192.168.137.100/24 scope global secondary ens33:1

ping测试，vip漂移时是否断开
[root@node2 ~]# ping 192.168.137.100
PING 192.168.137.100 (192.168.137.100) 56(84) bytes of data.
64 bytes from 192.168.137.100: icmp_seq=1 ttl=64 time=0.245 ms
64 bytes from 192.168.137.100: icmp_seq=2 ttl=64 time=0.442 ms
64 bytes from 192.168.137.100: icmp_seq=3 ttl=64 time=0.251 ms

现在如果启动node3 主keepalived服务
[root@node3 ~]# systemctl start keepalived
[root@node3 ~]# ip addr list |grep inet
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 192.168.137.17/24 brd 192.168.137.255 scope global ens33
    inet 192.168.137.100/24 scope global secondary ens33:1


```

### 定义keepalived 脚本

```
[root@node3 ~]# vim /etc/keepalived/notify.sh
#!/bin/bash
#
contact='root@localhost'
notify() {
        mailsubject="$(hostname) to be $1,vip floating"
        mailbody="$(date +'%F %T'): vrrp transition, $(hostname) changed to be $1"
        echo "$mailbody" |mail -s "$mailsubject" $contact
}
case $1 in
master)
        notify master
        ;;
backup)
        notify backup
        ;;
fault)
        notify fault
        ;;
*)
        echo "Usage: $(basename $0) {master|backup|fault}"
        exit 1
        ;;
esac
 
[root@node3 ~]# chmod +x /etc/keepalived/notify.sh 
[root@node3 ~]# scp /etc/keepalived/notify.sh 192.168.137.27:/etc/keepalived/
notify.sh                                                                                                      100%  391   543.6KB/s   00:00    

[root@node3 ~]# /etc/keepalived/notify.sh master
[root@node3 ~]# mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 1 message 1 new
>N  1 root                  Thu Feb 11 19:17  18/680   "node3 to be master,vip floating"
& 1
Message  1:
From root@node3.localdomain  Thu Feb 11 19:17:36 2021
Return-Path: <root@node3.localdomain>
X-Original-To: root@localhost
Delivered-To: root@localhost.localdomain
Date: Thu, 11 Feb 2021 19:17:36 -0500
To: root@localhost.localdomain
Subject: node3 to be master,vip floating
User-Agent: Heirloom mailx 12.5 7/5/10
Content-Type: text/plain; charset=us-ascii
From: root@node3.localdomain (root)
Status: R

2021-02-11 19:17:36: vrrp transition, node3 changed to be master

```

### 配置文件种加入配置项

```
加入最后三行，并且node4 也需要加入后面三行
[root@node3 ~]# vim /etc/keepalived/keepalived.conf
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.137.100/24 dev ens33 label ens33:1
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault  "/etc/keepalived/notify.sh fault"
}

```

### 测试结果

```
[root@node3 ~]# systemctl restart keepalived
[root@node3 ~]# mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 2 messages 1 new
    1 root                  Thu Feb 11 19:32  19/691   "node3 to be master,vip floating"
>N  2 root                  Thu Feb 11 19:38  18/680   "node3 to be master,vip floating"

[root@node4 ~]# systemctl restart keepalived
[root@node4 ~]# mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 3 messages 2 new 3 unread
 U  1 root                  Thu Feb 11 19:38  19/690   "node4 to be backup,vip floating"
>N  2 root                  Thu Feb 11 19:38  18/680   "node4 to be master,vip floating"
 N  3 root                  Thu Feb 11 19:38  18/680   "node4 to be backup,vip floating"

```

### 自定义日志

```
[root@node3 ~]# vim /etc/sysconfig/keepalived
KEEPALIVED_OPTIONS="-D -S 6"
[root@node3 ~]# vim /etc/rsyslog.conf 
local6.*                                                /var/log/keepalived.log
重启服务
[root@node3 ~]# systemctl restart rsyslog keepalived
[root@node3 ~]# ll /var/log/keepalived.log 
-rw-------. 1 root root 1637 Feb 11 21:15 /var/log/keepalived.log


```

### 开通QQ邮箱接收邮件

```
[root@node3 ~]# cat .mailrc 
set from=958678109@qq.com
set smtp=smtp.qq.com
set smtp-auth-user=958678109@qq.com
set smtp-auth-password=vmllkkqvyxgkbcbe
set smtp-auth=login
set ssl-verify=ignore

测试
[root@node3 ~]# echo test |mail -s linux37 958678109@qq.com

如果实现自动接收需要修改配置文件的邮箱接收地址和脚本文件的邮箱接收地址
[root@node3 ~]# vim /etc/keepalived/keepalived.conf
global_defs {
   notification_email {
        958678109@qq.com
   }

[root@node3 ~]# vim /etc/keepalived/notify.sh 
#!/bin/bash
#
contact='958678109@qq.com'
notify() {


```

### 双主模型

```

配置文件
[root@node3 ~]# cat /etc/keepalived/keepalived.conf
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.137.100/24 dev ens33 label ens33:1
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault  "/etc/keepalived/notify.sh fault"
}


virtual_server 192.168.137.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    protocol TCP
    sorry_server 127.0.0.1 80
    real_server 192.168.137.37 80 {
        weight 1
        HTTP_GET {
            url {
              path /
	      status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.137.47 80 {
        weight 1
        HTTP_GET {
            url {
              path /
	      status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
vrrp_instance VI_2 {
    state BACKUP
    interface ens33
    virtual_router_id 52
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.137.200/24 dev ens33 label ens33:1
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault  "/etc/keepalived/notify.sh fault"
}


virtual_server 192.168.137.200 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    protocol TCP
    sorry_server 127.0.0.1 80
    real_server 192.168.137.37 80 {
        weight 1
        HTTP_GET {
            url {
              path /
	      status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.137.47 80 {
        weight 1
        HTTP_GET {
            url {
              path /
	      status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}




验证：
[root@client ~]# 
[root@client ~]# while true;do curl 192.168.137.100;sleep 0.5;done
<h1>node6</h1>
<h1>node5</h1>
<h1>node6</h1>
^C
[root@client ~]# while true;do curl 192.168.137.200;sleep 0.5;done
sorry_server2
sorry_server2
sorry_server2

```

### 定义ivps规则

```
virtual_server 192.168.137.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    protocol TCP
    sorry_server 127.0.0.1 80
    real_server 192.168.137.37 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.137.47 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

启动rs1和rs2服务脚本
[root@node5 ~]# cat lvs_dr_rs.sh 
#!/bin/bash
#Author:wangxiaochun
#Date:2017-08-13
vip=192.168.137.100
mask='255.255.255.255'
dev=lo:1
rpm -q httpd &> /dev/null || yum -y install httpd &>/dev/null
service httpd start &> /dev/null && echo "The httpd Server is Ready!"
echo "<h1>`hostname`</h1>" > /var/www/html/index.html

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
[root@node5 ~]# bash lvs_dr_rs.sh start

安装ipvsadm服务
[root@node3 ~]#  yum -y install ipvsadm

重新启动lvs1 和llvs2 的keepalived 服务器
[root@node3 ~]# systemctl restart keepalived

验证：
[root@node3 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.137.100:80 rr
  -> 192.168.137.37:80            Route   1      0          0         
  -> 192.168.137.47:80            Route   1      0          0     
  
 客户端访问：
 [root@client ~]# !whi
while true;do curl 192.168.137.100;sleep 0.5;done
<h1>node6</h1>
<h1>node5</h1>
<h1>node6</h1>
<h1>node5</h1>
<h1>node6</h1>
<h1>node5</h1>
<h1>node6</h1>


定义sorry_server服务
lvs1和lvs2 都要安装httpd服务并且启动
[root@node3 ~]# yum -y install httpd
[root@node3 ~]# echo sorry_server1 > /var/www/html/index.html
[root@node3 ~]# systemctl start httpd

后端的rs1和rs2 都停止服务
[root@node5 ~]# service httpd stop

验证结果：
[root@client ~]# 
[root@client ~]# while true;do curl 192.168.137.100;sleep 0.5;done
sorry_server1
sorry_server1
sorry_server1
sorry_server1


```

总结：网关需要配置