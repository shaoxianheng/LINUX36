## 文本处理三剑客之awk

```
[root@node1 ~]# ll `which awk`
lrwxrwxrwx. 1 root root 4 Aug 27 20:52 /usr/bin/awk -> gawk
[root@node1 ~]# 

```

## 打印fstab文件的第一行

```
[root@node1 ~]# awk '{print $1}' /etc/fstab 

#
#
#
#
#
#
#
/dev/mapper/centos_node1-root
UUID=f7d1f48d-3bc9-4ef5-9ddb-6f8d7c20f3aa
/dev/mapper/centos_node1-swap

```

## 按照fstab文件的行数打印 字符串

```
[root@node1 ~]# awk '{print "hello,awk"}' /etc/fstab 
hello,awk
hello,awk
hello,awk
hello,awk
hello,awk
hello,awk
hello,awk
hello,awk
hello,awk
hello,awk
hello,awk

```

## 打印数字可以不用引号

```
[root@node1 ~]# awk '{print 100}' /etc/fstab 
100
100
100
100
100
100
100
100
100
100
100
[root@node1 ~]# awk '{print hello}' /etc/fstab 




字符串不加双引号会被当成变量







```

## 计算100乘以20

```
[root@node1 ~]# awk 'BEGIN{print 100*20}'
2000

```

```
[root@node1 ~]# awk 'BEGIN{print "number"}{print 100*200}' /etc/fstab 
number
20000
20000
20000
20000
20000
20000
20000
20000
20000
20000
20000
[root@node1 ~]# awk 'BEGIN{print "number"}{print 100*200}END{print "end"}' /etc/fstab 
number
20000
20000
20000
20000
20000
20000
20000
20000
20000
20000
20000
end

```

```
[root@node1 ~]# df |awk '{print $5}'
Use%
29%
0%
0%
1%
0%
17%
1%
0%

```

```
[root@node1 ~]# awk '{print $1}' access_log  |sort|uniq -c |sort -nr|head -3
   4870 172.20.116.228
   3429 172.20.116.208
   2834 172.20.0.222
[root@node1 ~]# 

```

```
[root@node1 ~]# awk -F":" '{print $1,$3}' /etc/passwd
root 0
bin 1
daemon 2
adm 3
lp 4
sync 5
shutdown 6

```

```
[root@node1 ~]# awk -F: '{print $1 "\t" $3 }' /etc/passwd

```

```
[root@node1 ~]# awk -F: '{print $0}' /etc/passwd

```

```
[root@node1 ~]# rev /etc/passwd|awk -F: '{print $1}' |rev

```

## awk变量

```
[root@node1 ~]# awk -v FS=: '{print $1,$3}' /etc/passwd
root 0
bin 1
daemon 2
adm 3
lp 4
sync 5
shutdown 6

```

```
[root@node1 ~]# awk -v FS=":" '{print $1FS$3}' /etc/passwd
root:0
bin:1
daemon:2
adm:3
lp:4
sync:5
shutdown:6
halt:7

```

```
[root@node1 ~]# fs=":"
[root@node1 ~]# awk -v FS=$fs '{print $1FS$3}' /etc/passwd
root:0
bin:1
daemon:2
adm:3
lp:4
sync:5
shutdown:6

```

```
[root@node1 ~]# df |awk '{print $5}' 
Use%
29%
0%
0%
1%
0%
17%
1%
0%
[root@node1 ~]# df |awk '{print $5}' |awk -F% '{print $1}'
Use
29
0
0
1
0
17
1
0
[root@node1 ~]# 

```

```
[root@node1 ~]# df |awk -F" +|%" '{print $5}' 
Use
29
0
0
1
0
17
1
0
[root@node1 ~]# 

```

```
[root@node1 ~]# df |awk -F" +|%" '{print $1,$5}' 
Filesystem Use
/dev/mapper/centos_node1-root 29
devtmpfs 0
tmpfs 0
tmpfs 1
tmpfs 0
/dev/sda1 17
tmpfs 1
tmpfs 0
[root@node1 ~]# 

```

```
24/May/2018:11:56:44
24/May/2018:11:56:44
24/May/2018:11:56:44
24/May/2018:11:56:44
[root@node1 ~]# awk -F "[\[ ]" '{print $5}' access_log 

```

```
172.20.116.174 24/May/2018:11:56:43
172.20.116.174 24/May/2018:11:56:43
172.20.116.174 24/May/2018:11:56:44
172.20.116.174 24/May/2018:11:56:44
172.20.116.174 24/May/2018:11:56:44
172.20.116.174 24/May/2018:11:56:44
172.20.116.174 24/May/2018:11:56:44
[root@node1 ~]# awk -F "[[ ]" '{print $1,$5}' access_log 

```

```
172.20.116.174	24/May/2018:11:56:44
172.20.116.174	24/May/2018:11:56:44
172.20.116.174	24/May/2018:11:56:44
172.20.116.174	24/May/2018:11:56:44
[root@node1 ~]# awk -F "[[ ]" -v OFS="\t" '{print $1,$5}' access_log 

```

```
[root@node1 ~]# cat /data/awk 
a,b,c
d;e,f
ggg;h;xxx

```

```
[root@node1 ~]# awk -F "," -v RS=";" '{print $1}' /data/awk 
a
e
h
xxxa

```

```
[root@node1 ~]# awk -F "," -v RS=";" '{print $3}' /data/awk 
c


```

```
[root@node1 ~]# awk -F: -v ORS=";" '{print $1,$3}' /etc/passwd
```

```
7
7
7
7
[root@node1 ~]# 
[root@node1 ~]# 
[root@node1 ~]# awk -F: '{print NF}' /etc/passwd

```

```
[root@node1 ~]# awk -F: '{print $NF}' /etc/passwd

```

```
[root@node1 ~]# awk -F: '{print $(NF-1)}' /etc/passwd

```

```
[root@node1 ~]# ss -nt| awk -F " +|:" '{print $(NF-2)}'
Address
192.168.43.212
[root@node1 ~]# ss -nt
State       Recv-Q Send-Q                           Local Address:Port                                          Peer Address:Port              
ESTAB       0      52                               192.168.43.97:22                                          192.168.43.212:50720              
[root@node1 ~]# 

```

```
[root@node1 ~]# awk -F: '{print NR,$1}' /etc/passwd

```

```
[root@node1 ~]# awk -F "," -v RS=";" '{print NR,$1}' /data/awk 
1 a
2 e
3 h
4 xxx

```

```
[root@node1 ~]# awk 'END{print NR}' /etc/fstab 
11

```

```
[root@node1 ~]# awk -F "," -v FS=";" '{print NR,$3}' /data/awk 
1 
2 
3 xxx
[root@node1 ~]# 

```

```
[root@node1 ~]# awk -F: '{print NR,$1}' /etc/passwd /etc/shadow
1 root
2 bin

```

```
[root@node1 ~]# awk -F: '{print FNR,$1}' /etc/passwd /etc/shadow
1 root
2 bin
3 daemon
4 adm
5 lp
6 sync
7 shutdown

```

```
[root@node1 ~]# awk -F: '{print FILENAME,FNR,$1}' /etc/passwd /etc/shadow

```

```
[root@node1 ~]# awk -F: 'BEGIN{print ARGC}' /etc/passwd /etc/shadow
3

```

```
[root@node1 ~]# awk -F: 'BEGIN{print ARGC,ARGV[0],ARGV[1],ARGV[2]}' /etc/passwd /etc/shadow
3 awk /etc/passwd /etc/shadow
[root@node1 ~]# 

```

```
[root@node1 ~]# awk -v NAME="magedu" 'BEGIN{print NAME}' 
magedu
[root@node1 ~]# 

```

```
[root@node1 ~]# awk -v NAME="magedu" 'BEGIN{print NAME}'
magedu
[root@node1 ~]# awk -v NAME="magedu" 'BEGIN{NAME="mage";print NAME}'
mage
[root@node1 ~]# awk -v NAME="magedu" 'BEGIN{print NAME;NAME="mage"}'
magedu

[root@node1 ~]# awk -v NAME="magedu" 'BEGIN{print NAME;NAME="mage";print NAME}'
magedu
mage

```

```
[root@node1 ~]# awk '{print NAME;NAME="magedu"}' /etc/passwd

magedu
magedu
magedu
magedu
magedu
magedu
magedu

```

```
[root@node1 ~]# cat f1.awk 
{print $1,$3}
[root@node1 ~]# awk -F: -f f1.awk /etc/passwd

```

```
[root@node1 ~]# 
[root@node1 ~]# cat f2.grep 
root
mage
[root@node1 ~]# grep -f f2.grep /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
dockerroot-latest:x:991:986:Docker User:/var/lib/docker-latest:/sbin/nologin
dockerroot:x:990:985:Docker User:/var/lib/docker:/sbin/nologin

```

```
[root@node1 ~]# file -f file1
/etc/passwd: ASCII text
/etc/issue:  ASCII text
[root@node1 ~]# 

```

```
[root@node1 ~]# awk 'BEGIN{i=1;print i++}'
1
[root@node1 ~]# awk 'BEGIN{i=1;print ++i}'
2
[root@node1 ~]# awk -F: '$3 >= 1000' /etc/passwd
nfsnobody:x:65534:65534:Anonymous NFS User:/var/lib/nfs:/sbin/nologin
shaoxh:x:1000:1000:shaoxh:/home/shaoxh:/bin/bash

```

```
[root@node1 ~]# awk -F: '$1 ~ "^root"' /etc/passwd
root:x:0:0:root:/root:/bin/bash
[root@node1 ~]# 

```

```
[root@node1 ~]# awk -F: '$1 ~ "root"' /etc/passwd
root:x:0:0:root:/root:/bin/bash
dockerroot-latest:x:991:986:Docker User:/var/lib/docker-latest:/sbin/nologin
dockerroot:x:990:985:Docker User:/var/lib/docker:/sbin/nologin
[root@node1 ~]# 

```

```
[root@node1 ~]# awk '$0 ~ "^UUID"' /etc/fstab 
UUID=f7d1f48d-3bc9-4ef5-9ddb-6f8d7c20f3aa /boot                   xfs     defaults        0 0
[root@node1 ~]# 

```

```
[root@node1 ~]# awk -F: '$3 >100 && $3 <=1000' /etc/passwd
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
polkitd:x:999:997:User for polkitd:/:/sbin/nologin
colord:x:998:996:User for colord:/var/lib/colord:/sbin/nologin
abrt:x:173:173::/etc/abrt:/sbin/nologin
libstoragemgmt:x:997:995:daemon account for libstoragemgmt:/var/run/lsm:/sbin/nologin
saslauth:x:996:76:Saslauthd user:/run/saslauthd:/sbin/nologin
setroubleshoot:x:995:993::/var/lib/setroubleshoot:/sbin/nologin
rtkit:x:172:172:RealtimeKit:/proc:/sbin/nologi
```

```
[root@node1 ~]# awk -F: '! ($3 >100)' /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin

```

```
[root@node1 ~]# awk -F: '$3>=1000?USERNAME="common user":USERNAME="system user" {print USERNAME,$3}' /etc/passwd
system user 0

```

```
[root@node1 ~]# awk -F: '/^r/{print $1,$3}' /etc/passwd
root 0
rpc 32
rtkit 172
radvd 75
rpcuser 29

```

```
[root@node1 ~]# awk '/^UUID/' /etc/fstab 
UUID=f7d1f48d-3bc9-4ef5-9ddb-6f8d7c20f3aa /boot                   xfs     defaults        0 0
[root@node1 ~]# 

```

```
[root@node1 ~]# awk '/^r/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
rtkit:x:172:172:RealtimeKit:/proc:/sbin/nologin
radvd:x:75:75:radvd user:/:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
[root@node1 ~]# 

```

```
[root@node1 ~]# df |awk -F " +|%" '/^\/dev\/sd/{print $1,$5}' 
/dev/sda1 18

```

```
[root@node1 ~]# ifconfig ens33| awk '/netmask/{print $2}'
192.168.43.97

```

```
[root@node1 ~]# ss -nt| awk -F" +|:" '/ESTA/{print $(NF-2)}'
192.168.43.212

```

```
[root@node1 ~]# awk -v i="0" 'i' /etc/fstab
[root@node1 ~]# awk -v i="0" '!i' /etc/fstab

#
# /etc/fstab
# Created by anaconda on Thu Aug 27 20:50:19 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos_node1-root /                       xfs     defaults        0 0
UUID=f7d1f48d-3bc9-4ef5-9ddb-6f8d7c20f3aa /boot                   xfs     defaults        0 0
/dev/mapper/centos_node1-swap swap                    swap    d
```

```
[root@node1 ~]# awk -v i="" 'i' /etc/fstab 
[root@node1 ~]# 

```

```
[root@node1 ~]# seq 10 |awk 'i++'
2
3
4
5
6
7
8
9
10
[root@node1 ~]# seq 10 |awk '++i'
1
2
3
4
5
6
7
8
9
10

```

```
[root@node1 ~]# seq 10 |awk 'i=!i'
1
3
5
7
9

```

```
[root@node1 ~]# seq 10 |awk -v i=1 'i=!i'
2
4
6
8
10

```

```
[root@node1 ~]# seq 10 |awk '!(i=!i)'
2
4
6
8
10
[root@node1 ~]# 

```

```
[root@node1 ~]# seq 10 |awk 'NR>=3 && NR<=6'
3
4
5
6

```

```
[root@node1 ~]# ifconfig |awk 'NR==2'
        inet 192.168.43.97  netmask 255.255.255.0  broadcast 192.168.43.255
[root@node1 ~]# 

```

```
[root@node1 ~]# awk '/^b/,/^f/' /etc/passwd
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
[root@node1 ~]# 

```

```
[root@node1 ~]# awk -F: '{print $1; print $3}' /etc/passwd

```

```
[root@node1 ~]# seq 10 |awk '{if(NR%2==1)print $0}'
1
3
5
7
9

```

```
[root@node1 ~]# seq 10 |awk '{if(NR%2==1)print "奇数行";else {print "偶数行"}}'
奇数行
偶数行
奇数行
偶数行
奇数行
偶数行
奇数行
偶数行
奇数行
偶数行
[root@node1 ~]# 

```

```
[root@node1 ~]# df |awk -F " +|%" '/^\/dev\/sd/{if ($5>10)print $1,$5}'
/dev/sda1 18

```

```
[root@node1 ~]# awk '/linux16/{i=1;while(i<=NF){print $i,length($i);i++}}' /boot/grub2/grub.cfg
linux16 7
/vmlinuz-3.10.0-693.el7.x86_64 30
root=/dev/mapper/centos_node1-root 34
ro 2
crashkernel=auto 16
rd.lvm.lv=centos_node1/root 27

```

```
[root@node1 ~]# awk 'BEGIN{i=1;sum=0;while(i<=100){sum+=i;i++};print "sum="sum}'
sum=5050
[root@node1 ~]# 

```

```
[root@node1 ~]# awk 'BEGIN{for(i=1;i<=100;i++){sum+=i};print sum}'
5050
[root@node1 ~]# 

```

```
[root@node1 ~]# for ((i=1;i<=100;i++));do let sum+=i;done;echo $sum
5050
[root@node1 ~]# 

```

```
[root@node1 ~]# time seq -s+ 100000 |bc
5000050000

real	0m0.122s
user	0m0.112s
sys	0m0.019s
[root@node1 ~]# time awk 'BEGIN{sum=0;for (i=1;i<=100000;i++){sum+=i};print sum}'
5000050000

real	0m0.036s
user	0m0.030s
sys	0m0.005s


[root@node1 ~]# time (sum=0;for((i=1;i<=100000;i++));do let sum+=i;done;echo $sum)
5000050000

real	0m1.404s
user	0m1.354s
sys	0m0.049s
[root@node1 ~]# 

[root@node1 ~]# time awk 'BEGIN{i=1;sum=0;while(i<=100000){sum+=i;i++};print "sum="sum}'
sum=5000050000

real	0m0.036s
user	0m0.029s
sys	0m0.007s


```

```
[root@node1 ~]# awk 'BEGIN{for (i=0;i<10;i++){if(i==5)continue;print i}}'
0
1
2
3
4
6
7
8
9

```

```
[root@node1 ~]# awk 'BEGIN{for (i=0;i<10;i++){if(i==5)break;print i}}'
0
1
2
3
4
[root@node1 ~]# 
[root@node1 ~]# 

```

```
[root@node1 ~]# seq 10|awk '{if(NR==5)next;print $0}'
1
2
3
4
6
7
8
9
10
[root@node1 ~]# 

```

```
[root@node1 ~]# awk '!a[$0]++ ' awk1.txt 
a
b
aa
bb
cc
dd

```

```
[root@node1 ~]# awk '{!a[$0]++;print $0,a[$0]}' awk1.txt 
a 1
b 1
aa 1
bb 1
cc 1
aa 2
dd 1
a 2

```

```
[root@node1 ~]# awk 'BEGIN{title["ceo"]="mage";title["coo"]="zhangsir";title["cto"]="wange";for(i in title){print i,title[i]}}'
coo zhangsir
ceo mage
cto wange

```

```
[root@node1 ~]# awk '{ip[$1]++}END{for (i in ip){print i,ip[i]}}' access_log 
```

```
[root@node1 ~]# awk '{ip[$1]++}END{for (i in ip){print i,ip[i]}}' access_log |sort -k2 -nr|head -3
172.20.116.228 4870
172.20.116.208 3429
172.20.0.222 2834

```

```
[root@node1 ~]# ss -tnl | awk '! /^State/{state[$1]++}END{for(i in state){print i,state[i]}}' 
LISTEN 11

```

```
[root@node1 ~]# awk -F" +|:" '/^ESTAB/{ip[$(NF-2)]++}END{for(i in ip)print i,ip[i]}' ss.log 
66.249.66.158 1
202.100.205.245 1
106.38.128.110 6
144.76.4.41 2
210.21.36.228 6
40.77.189.81 1
180.174.58.237 1
45.62.217.224 6
116.255.176.86 1
124.64.18.135 8
127.0.0.1 44
101.200.188.230 1
61.151.178.165 1
113.234.28.244 10
116.227.232.42 5
117.136.38.134 1
112.33.255.134 1
61.149.193.234 1
5.9.70.117 1
120.194.3.98 1
218.2.216.30 1
100.100.110.61 1
66.249.66.129 1
5.188.210.39 1
59.46.213.114 4
61.136.204.138 1

```

```
[root@node1 ~]# awk -F"[ .]" '{print $2}' list.txt >>list.txt 
[root@node1 ~]# cat list.txt 
1 www.magedu.com
2 blog.magedu.com
3 study.magedu.com
4 ke.magedu.com
www
blog
study
ke
[root@node1 ~]# 

```

```
[root@node1 ~]# awk -F"," '{i=1;max=$i;min=$i;while(i<=NF){if($i>max)max=$i;else if($i<min)min=$i;i++}}END{print "min="min,"max="max}' awktest1.t
xt min=557 max=32756

```

```
[root@node1 ~]# awk -F"/" '{fqdn[$3]++}END{for (i in  fqdn) print fqdn[i],i}' test2.log |sort -nr
2 www.magedu.com
2 blog.magedu.com
1 study.magedu.com
1 mail.magedu.com

```

```
[root@node1 ~]# awk 'NR!=1{sum[$2]+=$3;num[$2]++}END{for(i in sum){print i,sum[i]/num[i]}}' score.txt 
m 90
f 98.5
[root@node1 ~]# cat score.txt
name gender score
a m 100
b f 99
c f 98
d m 80
[root@node1 ~]# 

```

```
[root@node1 ~]# awk 'BEGIN{print rand()}'
0.237788
[root@node1 ~]# awk 'BEGIN{srand();print rand()}'
0.624439
[root@node1 ~]# awk 'BEGIN{srand();print int(rand()*100)}'
49
[root@node1 ~]# 

```

 