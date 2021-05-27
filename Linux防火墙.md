# Linux防火墙

## 查看内核中的iptables

```
[root@node1 smb]# grep -i iptables /boot/config-3.10.0-693.el7.x86_64 
CONFIG_IP_NF_IPTABLES=m
CONFIG_IP6_NF_IPTABLES=m
# iptables trigger is under Netfilter config (LED target)

```

## 拒绝112主机ping

```
[root@node1 ~]# iptables -vnL
Chain INPUT (policy ACCEPT 56 packets, 4209 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 37 packets, 3436 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@node1 ~]# iptables -t filter -A INPUT -s 192.168.0.112 -j DROP

[root@node2 ~]# ping 192.168.0.111
PING 192.168.0.111 (192.168.0.111) 56(84) bytes of data.

```

## tcpdump工具检测



## iptable拒绝网段

```
[root@node1 ~]# iptables -t filter -A INPUT -s 192.168.0.0/24 -j DROP
[root@node1 ~]# iptables -t filter -I INPUT -s 192.168.0.102/24 -j ACCEPT
[root@node1 ~]# iptables -vnL --line-number
Chain INPUT (policy ACCEPT 1 packets, 76 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1      303 22521 ACCEPT     all  --  *      *       192.168.0.0/24       0.0.0.0/0           
2       46  5831 DROP       all  --  *      *       192.168.0.0/24       0.0.0.0/0          
```

## 删除规则

```
[root@node1 ~]# iptables -t filter -D INPUT 2
[root@node1 ~]# iptables -vnL --line-number

```

## 计划任务规则

```
[root@node1 ~]# echo wall waring |at now+1 minutes
job 1 at Sat Jan 23 01:13:00 2021

```

## 加塞规则

```
[root@node1 ~]# iptables -t filter -A INPUT -s 192.168.0.112 -j DROP
[root@node1 ~]# iptables -t filter -A INPUT -s 192.168.0.113 -j DROP
[root@node1 ~]# iptables -t filter -I INPUT 2 -s 192.168.0.114 -j DROP
[root@node1 ~]# iptables -vnL --line-number
Chain INPUT (policy ACCEPT 55 packets, 3709 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 DROP       all  --  *      *       192.168.0.112        0.0.0.0/0           
2        0     0 DROP       all  --  *      *       192.168.0.114        0.0.0.0/0           
3        0     0 DROP       all  --  *      *       192.168.0.113        0.0.0.0/0  
```

## 替换规则

```
[root@node1 ~]# iptables -t filter -R INPUT 2 -s 192.168.0.115 -j DROP
[root@node1 ~]# iptables -vnL --line-number
Chain INPUT (policy ACCEPT 6 packets, 396 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 DROP       all  --  *      *       192.168.0.112        0.0.0.0/0           
2        0     0 DROP       all  --  *      *       192.168.0.115        0.0.0.0/0           
3        0     0 DROP       all  --  *      *       192.168.0.113        0.0.0.0/0  
```

## 清零计数器

```
[root@node1 ~]# iptables -vnL --line-number
Chain INPUT (policy ACCEPT 75 packets, 5110 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1       13  1092 DROP       all  --  *      *       192.168.0.112        0.0.0.0/0   

[root@node1 ~]# iptables -Z INPUT
[root@node1 ~]# iptables -vnL --line-number
Chain INPUT (policy ACCEPT 6 packets, 396 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 DROP       all  --  *      *       192.168.0.112        0.0.0.0/0           


```

## 可以写多个地址

```
[root@node1 ~]# iptables -t filter -A INPUT -s 192.168.0.112,192.168.0.113 -j DROP
[root@node1 ~]# iptables -vnL
Chain INPUT (policy ACCEPT 28 packets, 1893 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  *      *       192.168.0.112        0.0.0.0/0           
    0     0 DROP       all  --  *      *       192.168.0.113        0.0.0.0/0           


```

## 拒绝所有

```
[root@node1 ~]# iptables -A INPUT -j REJECT

```

## 指定网卡

```
[root@node1 ~]# iptables -A INPUT -i lo -j ACCEPT
[root@node1 ~]# iptables -vnL
Chain INPUT (policy ACCEPT 22 packets, 1452 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           

```

## 拒绝出去的

```
[root@node1 ~]# iptables -A OUTPUT -j REJECT

```

## 默认策略为DROP

```
[root@node1 ~]# iptables -P INPUT DROP

```

## uptables 命令扩展

```
安装httpd和mysql
[root@node1 ~]# yum -y install httpd mariadb-server

启动httpd和mysql
[root@node1 ~]# systemctl start httpd mariadb

配置http页面
[root@node1 ~]# echo welcome to magedu > /va/www/html/index.html

添加iptables 规则 允许本机和宿主机 拒绝其他
[root@node1 ~]# iptables -A INPUT -s 192.168.0.111,192.168.0.104 -j ACCEPT
[root@node1 ~]# iptables -A INPUT -j REJECT
[root@node1 ~]# curl 192.168.0.111
welcome to magedu

112 主机测试结果
[root@node2 ~]# curl 192.168.0.111
curl: (7) Failed connect to 192.168.0.111:80; Connection refused

允许112主机 访问80端口
[root@node1 ~]# iptables -I INPUT 3 -s 192.168.0.112 -p tcp --dport 80 -j ACCEPT
[root@node2 ~]# curl 192.168.0.111
welcome to magedu

允许112 访问3306端口
[root@node1 ~]# iptables -I INPUT 3 -s 192.168.0.112 -p tcp --dport 3306 -j ACCEPT
启动mysql
[root@node1 ~]# systemctl start mariadb

授权用户
[root@node1 ~]# mysql -e "grant all on *.* to test@'192.168.0.%' identified by 'shaoxh'"

测试连接
[root@node2 ~]# mysql -h192.168.0.111 -utest -pshaoxh


```

## 扩展选项-标记位

```
[root@node1 ~]# iptables -I INPUT 5 -s 192.168.0.113 -p tcp --syn -j REJECT

```

##  	可以ping其他主机，但是不能别人ping自己

```
[root@node1 ~]# iptables -I INPUT 5 -p icmp --icmp-type 0 -j ACCEPT
[root@node1 ~]# ping 192.168.0.112

```

## 显示模块使用

### 启动samba服务

```
[root@node1 ~]# iptables -F
[root@node1 ~]#  systemctl start smb
拒绝139 445 端口
[root@node1 ~]# iptables -A INPUT  -p tcp -m multiport --dport 139,445 -j REJECT
[root@node1 ~]# smbpasswd -a shaoxh
New SMB password:
Retype new SMB password:

客户端验证：
[root@node3 ~]# smbclient //192.168.0.111/homes -U shaoxh%shaoxh
Connection to 192.168.0.111 failed (Error NT_STATUS_CONNECTION_REFUSED)

```

### 使用mac模块

```
依据node3的mac拒绝访问
[root@node3 ~]# iptables -F
[root@node3 ~]# curl 192.168.0.111
welcome to magedu

规则定义
[root@node1 ~]# iptables -A INPUT -m mac --mac-source 00:0c:29:95:78:28 -j REJECT

验证
[root@node3 ~]# curl 192.168.0.111
curl: (7) Failed connect to 192.168.0.111:80; Connection refused


```

### string 扩展

```
[root@node1 ~]# iptables -F
[root@node1 ~]# cat /var/www/html/test.html 
www.google.com

拒绝访问内容有google字符串
[root@node1 ~]# iptables -A OUTPUT -p tcp --sport 80 -m string --algo bm --string "google" -j REJECT

验证
[root@node3 ~]# curl 192.168.0.111/test.html

[root@node3 ~]# curl 192.168.0.111/index.html
welcome to magedu
[root@node3 ~]# 


```

### time扩展

```
早上9点和晚上18点不能上网  UTC计算 本地时间要减去8
[root@node1 ~]# iptables -A INPUT -m time --timestart 1:00 --timestop 10:00 -j REJECT

```

### connlimit扩展

```
测试
[root@node2 ~]# gcc flood_connect.c -o flood
[root@node2 ~]# ./flood 192.168.0.111
Starting flood connect attack on 192.168.0.111 port 80

发起请求
[root@node3 ~]# curl 192.168.0.111

查看并发连接数
[root@node1 ~]# ss -tn

拒绝每个ip并发连接数大于100的连接
[root@node1 ~]# iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 100 -j REJECT

```

### limit 扩展

```
基于收发报文的速率做匹配
令牌桶过滤器
--limit #[/second|/minute|/hour|/day]
--limit-burst number
 [root@node1 ~]# iptables -A INPUT -p icmp --icmp-type 8 -m limit --limit 20/minute --limit-burst 10 -j ACCEPT
 [root@node1 ~]# iptables -A INPUT -j REJECT

```

### state扩展

```
拒绝new状态连接
[root@node1 ~]# iptables -A INPUT -p tcp --dport 22 -m state --state new -j REJECT

验证：
[root@node2 ~]# ssh 192.168.0.111
ssh: connect to host 192.168.0.111 port 22: Connection refused

```

```
[root@node1 ~]# iptables -A INPUT -s 127.0.0.1 -j ACCEPT
[root@node1 ~]# iptables -A INPUT -s 192.168.0.104 -j ACCEPT
[root@node1 ~]# yum -y install vsftpd
[root@node1 ~]# systemctl start vsftpd
[root@node1 ~]# iptables -A INPUT -j REJECT

验证
[root@node2 ~]# ftp 192.168.0.111
Name (192.168.0.111:root): ftp
227 Entering Passive Mode (192,168,0,111,121,238).
ftp: connect: Connection refused

加载跟踪模块
[root@node1 ~]# modprobe nf_conntrack_ftp
[root@node1 ~]# iptables -I INPUT 3 -m state --state ESTABLISHED,RELATED -j ACCEPT

验证
[root@node2 ~]# ftp 192.168.0.111
Connected to 192.168.0.111 (192.168.0.111).
220 (vsFTPd 3.0.2)
Name (192.168.0.111:root): ftp
ftp> ls
drwxr-xr-x    2 0        0               6 Oct 13 16:10 pub
226 Directory send OK.
```

## TARGET

### log

```
[root@node1 ~]# iptables -D INPUT 4
[root@node1 ~]# iptables -I INPUT 4 -p icmp -j ACCEPT

ping主机node1
[root@node2 ~]# ping 192.168.0.111
PING 192.168.0.111 (192.168.0.111) 56(84) bytes of data.
64 bytes from 192.168.0.111: icmp_seq=1 ttl=64 time=0.240 ms
64 bytes from 192.168.0.111: icmp_seq=2 ttl=64 time=0.332 ms

[root@node1 ~]# iptables -D INPUT 4
ping 命令不会中断，如果ctrl+C停止ping ，再次发起ping请求，就不能ping通

生成日志规则
[root@node1 ~]# iptables -I INPUT 3 -s 192.168.0.112 -j LOG --log-prefix "from 0.112 access:"
监控日志
[root@node1 ~]# tail -f /var/log/messages 

验证
[root@node2 ~]# ping 192.168.0.111

```

## 保存规则

```
iptables-save > iptables.rule
iptables-restore < iptables.rule 
vim /etc/rc.d/rc.local
iptables-restore < /root/iptables.rule 
chmod +x /etc/rc.d/rc.local

```

## 安装iptables-services

```
[root@node1 ~]# yum -y install iptables-service
[root@node1 ~]# systemctl stop firewalld
[root@node1 ~]# systemctl status iptables
[root@node1 ~]# iptables -F
[root@node1 ~]# systemctl start iptables.service 
[root@node1 ~]# iptables -vnL

```

## SNAT原理

```
环境规划
node1                    node2               node3
none                     dhcp                none
nat模式                   nat+桥接             桥接模式
192.168.38.128           192.168.38.129       192.168.0.110
                         192.168.0.111
GW：                                          GW:
192.168.38.129                                192.168.0.111

node1和node3可以互相ping通
[root@node2 ~]# cat /etc/sysctl.conf 
net.ipv4.ip_forward=1
[root@node2 ~]# sysctl -p
net.ipv4.ip_forward = 1

验证：
[root@node1 ~]# !ping
ping 192.168.0.110
PING 192.168.0.110 (192.168.0.110) 56(84) bytes of data.
64 bytes from 192.168.0.110: icmp_seq=1 ttl=63 time=0.354 ms
64 bytes from 192.168.0.110: icmp_seq=2 ttl=63 time=0.306 ms

[root@node3 ~]# !ping
ping 192.168.38.128
PING 192.168.38.128 (192.168.38.128) 56(84) bytes of data.
64 bytes from 192.168.38.128: icmp_seq=1 ttl=63 time=0.581 ms
64 bytes from 192.168.38.128: icmp_seq=2 ttl=63 time=0.410 ms

假设node1网段为内网，node3网段为外网
使node1可以ping外网，外网不能ping内网
在转发上拒绝所有
[root@node2 ~]# iptables -A FORWARD -j REJECT
请求报文
[root@node2 ~]# iptables -I FORWARD 1 -s 192.168.38.0/24 -p icmp --icmp-type 8 -j ACCEPT
响应报文
[root@node2 ~]# iptables -I FORWARD 1 -d 192.168.38.0/24 -p icmp --icmp-type 0 -j ACCEPT

验证
[root@node1 ~]# ping 192.168.0.110
PING 192.168.0.110 (192.168.0.110) 56(84) bytes of data.
64 bytes from 192.168.0.110: icmp_seq=1 ttl=63 time=0.712 ms

[root@node3 ~]# ping 192.168.38.128
PING 192.168.38.128 (192.168.38.128) 56(84) bytes of data.
From 192.168.0.111 icmp_seq=1 Destination Port Unreachable

第二种状态跟踪
[root@node2 ~]# iptables -D FORWARD 1
[root@node2 ~]# iptables -I FORWARD 1 -m state --state ESTABLISHED,RELATED -j ACCEPT

验证：也可以ping通

内网可以访问外网的httpd服务：
[root@node3 yum.repos.d]# systemctl start httpd
[root@node3 yum.repos.d]# echo internet server > /var/www/html/index.html
[root@node3 yum.repos.d]# curl localhost

添加httpd 的规则
[root@node2 ~]# iptables -I FORWARD 2 -s 192.168.38.0/24  -p tcp --dport 80  -j ACCEPT

验证
[root@node1 ~]# curl 192.168.0.110
internet server

80和443
[root@node2 ~]# iptables -R FORWARD 2 -s 192.168.38.0/24  -p tcp -m multiport  --dports 80,443  -j ACCEPT
[root@node3 yum.repos.d]# yum -y install mod_ssl
[root@node3 yum.repos.d]# systemctl restart httpd
[root@node3 yum.repos.d]# ss -tnl|grep  -E "443|80"
LISTEN     0      128         :::80                      :::*                  
LISTEN     0      128         :::443                     :::*    

验证
[root@node1 ~]# curl -k https://192.168.0.110
internet server

如果外网访问内网的httpd服务
[root@node2 ~]# iptables -I FORWARD 4 -d 192.168.38.128 -p tcp -m multiport --dports 80,443 -j ACCEPT

验证
[root@node3 yum.repos.d]# curl 192.168.38.128
hello world

```

## 自定义链

```
[root@node2 ~]# iptables -N TONET
[root@node2 ~]# iptables -A TONET -s 192.168.38.0/24 -p tcp -m multiport --dports 80,443 -j ACCEPT
[root@node2 ~]# iptables -A TONET -s 192.168.38.0/24 -p icmp --icmp-type 8 -j ACCEPT

删除原来FORWARD上的内网规则
[root@node2 ~]# iptables -D FORWARD 2
[root@node2 ~]# iptables -D FORWARD 2

加入自己定义的规则集
[root@node2 ~]# iptables -I FORWARD 2 -j TONET

允许外网的22端口
[root@node2 ~]# iptables -R TONET 1 -s 192.168.38.0/24 -p tcp -m multiport --dports 22,80,443 -j ACCEPT
验证
[root@node1 ~]# ssh 192.168.0.110
Are you sure you want to continue connecting (yes/no)?

-N 创建
-X 删除
-E 修改

删除自定义链，先删除引用
[root@node2 ~]# iptables -D FORWARD 2
清空链
[root@node2 ~]# iptables -F TONET
删除链
[root@node2 ~]# iptables -X TONET


```

## NET

```
请求出去报文是源地址 是SNAT   请求出去的报文时目标地址 是DNAT
```

## 获取本地网路地址

```
[root@node2 ~]# curl http://ipinfo.io/ip
223.98.64.48[root@node2 ~]# 
[root@node2 ~]# 
[root@node2 ~]# 
[root@node2 ~]# 
[root@node2 ~]# curl ip.sb
223.98.64.48

[root@node2 ~]# curl ifconfig.me
223.98.64.48[root@node2 ~]# 

[root@node2 ~]# curl ifconfig.me/all.json
{
  "ip_addr": "223.98.64.48",
  "remote_host": "unavailable",
  "user_agent": "curl/7.29.0",
  "port": 39584,
  "method": "GET",
  "mime": "*/*",
  "via": "1.1 google",
  "forwarded": "223.98.64.48, 216.239.38.21"

```

