# Shell 编程进阶

##  关键字组合才能使用

```
[root@node1 tmpfiles.d]# type for
for is a shell keyword
[root@node1 tmpfiles.d]# type while
while is a shell keyword
[root@node1 tmpfiles.d]# type if
if is a shell keyword

```

## for循环

```
[root@node1 tmpfiles.d]# for Num in {1..10};do echo $Num;done
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
[root@node1 tmpfiles.d]# seq 1 2 10
1
3
5
7
9
[root@node1 tmpfiles.d]# seq 2 2 10
2
4
6
8
10
[root@node1 tmpfiles.d]# seq 10 -2 1
10
8
6
4
2

```

## 通配符的for循环

```
[root@node1 tmpfiles.d]# cd /data/
[root@node1 data]# touch f{1..10}txt
[root@node1 data]# ls
f10txt  f1txt  f2txt  f3txt  f4txt  f5txt  f6txt  f7txt  f8txt  f9txt

[root@node1 data]# for file in "*txt";do echo $file;done
f10txt f1txt f2txt f3txt f4txt f5txt f6txt f7txt f8txt f9txt
[root@node1 data]# for file in *txt;do echo $file;done
f10txt
f1txt
f2txt
f3txt
f4txt
f5txt
f6txt
f7txt
f8txt
f9txt

```

## 位置参数

```
[root@node1 data]# cat for_test.sh 
#!/bin/bash
#
for i in $@;do
    echo arg is $i
done
[root@node1 data]# ./for_test.sh a b c
arg is a
arg is b
arg is c

```

## 1加到100

```
[root@node1 data]# seq -s+ 100|bc
5050

```

```
[root@node1 data]# cat sum.sh 
#!/bin/bash
declare -i sum=0
for i in {1..100}
do
    let sum+=i
done
echo sum=$sum
    

```

## 奇数之和

```
[root@node1 data]# cat ./sum.sh
#!/bin/bash
declare -i sum=0
for i in {1..100}
do
    if [ $[i%2] -eq 1 ]
    then
    let sum+=i
    fi
done
echo sum=$sum
    

```

## 创建10个用户

```
[root@node1 data]# cat user.sh 
#!/bin/bash
#
for i in {1..10}
do
    useradd user$i
    password=`tr -dc '[[:alnum:]]' < /dev/urandom |head -c 8`
    echo user$i:$password |tee -a pass.txt|chpasswd
    passwd -e user$i
    echo User$i is created
done
[root@node1 data]# 

```

## 删除用户

```
[root@node1 data]# for i in {1..10};do userdel -r user$i ;done

```

## 创建用户

```
[root@node1 data]# cat user2.sh 
#!/bin/bash
#
NAME="
alice
bob
tom
jack
"
for user in $NAME
do
    useradd $user
done

```

## 文件列表

```
[root@node1 data]# cat userlist.txt 
xiaoming
xiaohong
cuihua
suancai
[root@node1 data]# cat user3.sh 
#!/bin/bash
#
for user in `cat userlist.txt`
do
    useradd $user
done

```

## 正则表达式

```
[root@node1 data]# for i in `sed -nr '/^alice/,$s/^([^:]+):.*/\1/p' /etc/passwd`;do userdel -r $i;done
[root@node1 data]# sed -nr '/^alice/,$s/^([^:]+):.*/\1/p' /etc/passwd

```

```
[root@node1 data]# echo 1.html.tar|sed -rn 's/(.*)\.(.*)/\1/p'
1.html
[root@node1 data]# echo 1.html.tar|sed -rn 's/(.*)\.(.*)/\2/p'
tar
[root@node1 data]# 

```

```
[root@node1 data]# echo 1.html.tar|sed -rn 's/(.*)\.(.*)/\1/p'
1.html
[root@node1 data]# echo 1.html.tar|sed -rn 's/(.*)\.(.*)/\2/p'
tar
[root@node1 data]# 

```

## 批量修改文件的名字

```
[root@node1 home]# ls /data/test/
1.a  2.b  3.c
[root@node1 home]# ./rename.sh 
[root@node1 home]# ls /data/test/
1.abc  2.abc  3.abc
[root@node1 home]# cat rename.sh 
#!/bin/bash
#
cd /data/test
for i in *
do
    mv $i $(echo $i|sed -r 's/(^.*[.])([^.]*$)/\1abc/')
done
[root@node1 home]# 

```

```
[root@node1 test]# ls
1.a  2.b  3.c
[root@node1 test]# vim /home/rename.sh 
[root@node1 test]# chmod +x /home/rename.sh
[root@node1 test]# bash /home/rename.sh 
[root@node1 test]# ls
1.abc  2.abc  3.abc
[root@node1 test]# cat /home/rename.sh 
#!/bin/bash
#
Dir=/data/test
cd $Dir
for file in * 
do
    filename=`echo $file |sed -nr 's@(.*)\..*$@\1.abc@p'`
    mv $file $filename
done

```

```
[root@node1 ~]# cat rename3.sh 
#!/bin/bash
#
set -u
set -e
Dir=/data/test
cd $Dir
for file in *
do
    file_name=`echo $file | sed -nr 's#(.*)\..*$#\1#p'`
    mv $file $file_name.abc
    echo $file is rename $file_name.abc
done

```

## 并行扫描网段

```
[root@node1 ~]# cat netscan.sh 
#!/bin/bash
#
NET=192.168.1
for HOSTID in {1..254}
do {
    if ping -c1 -W1 $NET.$HOSTID &> /dev/null 
    then
        echo $NET.$HOSTID is up |tee -a hostlist.txt
    fi
   } &
done
wait

```

## 打印机矩形

```
[root@node1 ~]# cat ./for_for.sh
for i in {1..10};do
    for j in {1..10};do
        echo -e '*\c'
    done
    echo 
done

```

## 打印星星

```
[root@node1 ~]# cat ./for_for.sh 
#!/bin/bash
#
read -p "please input row:" ROW
read -p "please input column" COL
COLOR="\033["

for i in `seq $ROW`;do
    for j in `seq $COL`;do
    echo -e "${COLOR}$[RANDOM%7+31];1;5m*\033[0m\c"
    done
    echo
done

```

## 99乘法表

```
[root@node1 ~]# cat for_9X9.sh 
#!/bin/bash
#
for i in {1..9};do
    for j in `seq $i`;do
        echo -e "${j}X$i=$[j*i]\t\c"
        done
        echo
done

```

## C风格的写法

```
[root@node1 ~]# cat ./sum_1.sh 
#!/bin/bash
#
sum=0
for ((i=1;i<=100;i++));do
    let sum+=i
done
echo sum=$sum

```

```
[root@node1 ~]# cat sum_1.sh 
#!/bin/bash
#
#sum=0
for ((sum=0,i=1;i<=100;i++));do
    let sum+=i
done
echo sum=$sum

```

## 小括号里可以不加上$

```
[root@node1 ~]# cat ./sum_1.sh
#!/bin/bash
#
#sum=0
n=10
for ((sum=0,i=1;i<=n;i++));do
    let sum+=i
done
echo sum=$sum

```

## C风格99

```
[root@node1 ~]# cat ./for_9X9.sh 
#!/bin/bash
#
for i in {1..9};do
    for j in `seq $i`;do
        echo -e "${j}X$i=$[j*i]\t\c"
        done
        echo
done

for ((i=1;i<=9;i++));do
    for ((j=1;j<=i;j++));do
        echo -e "${j}X$i=$[j*i] \t\c"
        done
        echo 
done

```

https://blog.csdn.net/hongrisl/article/details/88062228

## 打印国际棋盘

```

[root@node1 ~]# cat qipan.sh 
#!/bin/bash
    
for i in {1..8};do
        temp1=$[ $i % 2 ]
    
        for j in {1..8};do
        temp2=$[ $j % 2 ]
 
        if [ $temp1 -eq  $temp2  ];then
                echo -e -n "\033[47m  \033[0m"
        else
                echo -e -n "\033[41m  \033[0m"
        fi
 
        done
 
        echo 
done

```

## while循环比较最大最小值

```
[root@node1 ~]# cat ./while_max_mix.sh
#!/bin/bash
#
i=1
MAX=$RANDOM
MIN=$RANDOM
echo $RANDOM
while [ $i -lt 10 ];do
    N=$RANDOM
    echo $N
    if [ $N -gt $MAX ];then
        MAX=$N
    elif [ $N -lt $MIN ];then
        MIN=$N
    fi 
    let i++
done
echo MAX=$MAX,MIN=$MIN

```

## php-fpm

```
[root@node1 ~]# systemctl start php-fpm
[root@node1 ~]# ss -tnl
State       Recv-Q Send-Q                           Local Address:Port                                          Peer Address:Port              
LISTEN      0      128                                          *:111                                                      *:*                  
LISTEN      0      5                                192.168.122.1:53                                                       *:*                  
LISTEN      0      128                                          *:22                                                       *:*                  
LISTEN      0      128                                  127.0.0.1:631                                                      *:*                  
LISTEN      0      100                                  127.0.0.1:25                                                       *:*                  
LISTEN      0      128                                  127.0.0.1:6010                                                     *:*                  
LISTEN      0      128                                  127.0.0.1:9000                                                     *:*                  
LISTEN      0      128                                         :::111                                                     :::*                  
LISTEN      0      128                                         :::22                                                      :::*                  
LISTEN      0      128                                        ::1:631                                                     :::*                  
LISTEN      0      100                                        ::1:25                                                      :::*                  
LISTEN      0      128                                        ::1:6010        
```

## 查看php-fpm状态

```
[root@node1 ~]# killall php-fpm
[root@node1 ~]# killall -0 php-fpm
php-fpm: no process found
[root@node1 ~]# echo $?
1
[root@node1 ~]# systemctl restart php-fpm
[root@node1 ~]# killall -0 php-fpm
[root@node1 ~]# echo $?
0

```

## 脚本监控php-fpm的状态

```
[root@node1 ~]# cat ./while_php-fpm_status.sh
#!/bin/bash
#
SLEEPTIME=10
while :;do
    if killall -0 php-fpm &> /dev/null;then
       :
    else
        systemctl restart php-fpm &> /dev/null
        echo At `date +%F-%T` php-fpm is restart >> /var/log/monitor.log
    fi
    sleep $SLEEPTIME
done

```

## continue脚本

```
[root@node1 ~]# ./continue.sh 
i=0
i=1
i=2
i=3
i=4
i=6
i=7
i=8
i=9
[root@node1 ~]# cat ./continue.sh
#!/bin/bash
#
for ((i=0;i<10;i++));do
    if [ $i -eq 5 ];then
        continue
    fi
    echo i=$i
done

```



## break脚本

```
[root@node1 ~]# ./break.sh 
i=0
i=1
i=2
i=3
i=4
finish
[root@node1 ~]# cat ./break.sh
#!/bin/bash
#
for ((i=0;i<10;i++));do
    if [ $i -eq 5 ];then
        break
    fi
    echo i=$i
done
echo finish

```

## continue2.sh

```
[root@node1 ~]# cat ./continue2.sh 
#!/bin/bash
#
for ((i=0;i<10;i++));do
    for ((j=0;j<10;j++));do
        if [ $j -eq 5 ];then
            continue 2
        fi
	echo j=$j
    done
    echo i=$i
done
echo finish


```

## break2.sh

```
[root@node1 ~]# cat ./break2.sh
#!/bin/bash
#
for ((i=0;i<10;i++));do
    for ((j=0;j<10;j++));do
        if [ $j -eq 5 ];then
            break 2
        fi
        echo j=$j
    done
    echo i=$i
done
echo finish

```

## 菜单选项

```
[root@node1 ~]# cat ./menu.sh 
#!/bin/bash
#
cat << EOF
1) gongbaojiding
2) beijingkaoya
3) fotiaoqiang
4) haishen
5) baoyu
6) quit
EOF
while read -p "Please choose the number:" NUM;do
case $NUM in
1)
    echo gongbaojiding price is ￥30
    ;;
2)
    echo beijingkaoya price ￥80
    ;;
3)
    echo fotiaoqiang price ￥200
    ;;
4)
    echo haishen price ￥10
    ;;
5) 
    echo baoyu price ￥50
    ;;
6)
    break
    ;;
*)
   echo "Please input again"
esac
done


```

shift

```
[root@node1 ~]# bash test.sh a b c d e f
1st arg is a
2st arg is b
all arg are a b c d e f
1st arg is d
2st arg is e
all arg are d e f
1st arg is e
2st arg is f
all arg are e f
[root@node1 ~]# 
[root@node1 ~]# 
[root@node1 ~]# cat test.sh 
#!/bin/bash
#
echo 1st arg is $1
echo 2st arg is $2
echo all arg are $@
shift 3

echo 1st arg is $1
echo 2st arg is $2
echo all arg are $@
shift

echo 1st arg is $1
echo 2st arg is $2
echo all arg are $@

```

## shift_user.sh

```
[root@node1 ~]# cat shift_user.sh 
#!/bin/bash
#
while [ "$1" ];do
    useradd $1
    echo user:$1 is created
    shift
done
echo finish



[root@node1 ~]# bash shift_user.sh a b c
user:a is created
user:b is created
user:c is created
finish

```

## 猜数字大小

```
[root@node1 ~]# cat ./guess.sh 
#!/bin/bash
#
N=$[RANDOM%10+1]
while read -p "input a number(1-10):" NUM;do
    if [[ $NUM =~ ^[[:digit:]]+$ ]];then
        if [ $NUM -eq $N ];then
            echo ok
            break
        elif [ $NUM -gt $N ];then
            echo too large!
    	else
	    echo too small!
        fi
    else
        echo "Please input a digit"
    fi
done

```

## 多个变量赋值

```
[root@node1 ~]# echo a b c | while read x y z ;do echo x=$x,y=$y,z=$z;done
x=a,y=b,z=c
[root@node1 ~]# echo a b c | read x y z ; echo x=$x,y=$y,z=$z
x=,y=,z=
[root@node1 ~]# echo "  a b c  " |while read x y z ;do echo x=$x,y=$y,z=$z;done
x=a,y=b,z=c
[root@node1 ~]# 

```

## 从文件读入值

```
[root@node1 ~]# cat f1.txt 
a b c
1 2 3
x y  zz
[root@node1 ~]# cat f1.txt |while read x y z ;do echo x=$x,y=$y,z=$z;done
x=a,y=b,z=c
x=1,y=2,z=3
x=x,y=y,z=zz

```

## 磁盘利用率

```
[root@node1 ~]# cat while_disk.sh 
#!/bin/bash
#
WARNING=15
df | sed -rn '/^\/dev\/sd/s#^([^[:space:]]+).* ([[:digit:]]+)%.*$#\1 \2#p'|while read part use;do
    if [ $use -gt $WARNING ];then
        echo $part will be full,use:$use
    fi
done

```

添加磁盘

```
[root@node1 ~]# echo "- - -" > /sys/class/scsi_host/host0/scan 
[root@node1 ~]# lsblk 
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                     8:0    0   20G  0 disk 
├─sda1                  8:1    0    1G  0 part /boot
└─sda2                  8:2    0   19G  0 part 
  ├─centos_node1-root 253:0    0   17G  0 lvm  /
  └─centos_node1-swap 253:1    0    2G  0 lvm  [SWAP]
sdb                     8:16   0   20G  0 disk 
sr0                    11:0    1  8.1G  0 rom  
```

```
[root@node1 ~]# mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=65536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

```

```
[root@node1 ~]# mount /dev/sdb1 /mnt/
[root@node1 ~]# dd if=/dev/zero of=/mnt/f1 bs=1M count=200
200+0 records in
200+0 records out
209715200 bytes (210 MB) copied, 0.872201 s, 240 MB/s
[root@node1 ~]# ./while_
while_disk.sh            while_max_mix.sh         while_php-fpm_status.sh  
[root@node1 ~]# ./while_disk.sh 
/dev/sda1 will be full,use:18
/dev/sdb1 will be full,use:23

```

## 记录日志访问ip次数

```
[root@node1 ~]# cat while_read_accesslog.sh 
#!/bin/bash
#
sed -rn 's/^([^[:space:]]+).*/\1/p' access_log |sort |uniq -c >iplist.txt
while read cont ip;do
    if [ $cont -gt 100 ];then
        echo from $ip access $cont  >> crack.log
    fi
done < iplist.txt

```

## lastb命令

```
[root@node1 ~]# lastb -f btmp-34期魏阳登录错误日志 

```

```
[root@node1 ~]# lastb -f btmp-34期魏阳登录错误日志 |sed -rn 's/.* ([0-9.]{7,15}) .*/\1/p'|sort |uniq -c > lastb.log

```

select 命令

```
[root@node1 ~]# PS3="Please input a number: ";select menu in gongbaojiding baoyu haishen longxia;do true;done
1) gongbaojiding
2) baoyu
3) haishen
4) longxia
Please input a number: 1
Please input a number: 2
Please input a number: 3
Please input a number: 

```

PS2的作用

```
[root@node1 ~]# cat << EOF
> abc
> EOF
abc
[root@node1 ~]# PS2=XXX
[root@node1 ~]# cat << EOF
XXXabc
XXXEOF
abc

```

## 菜单

```
[root@node1 ~]# bash select.sh 
1) gongbaojiding  3) fotiaoqiang    5) baoyu
2) kaoya	  4) haishen	    6) quit
Please input a number: 1
gongbaojiding price is ￥30
Please input a number: 6
[root@node1 ~]# cat select.sh 
#!/bin/bash
#
PS3="Please input a number: "
select menu in gongbaojiding kaoya fotiaoqiang haishen baoyu quit;do
    case $REPLY in
    1)
        echo $menu price is ￥30
        ;;
    2)
        echo $menu price ￥80
        ;;
    3)
        echo $menu price ￥200
        ;;
    4)
        echo $menu price ￥10
        ;;
    5)
        echo $menu price ￥50
        ;;
    6)
        break
        ;;
    *)
       echo "pleast input again"
    esac
done


```

## 函数

```
[root@node1 data]# 
[root@node1 data]# func_test () { echo test function ; }
[root@node1 data]# func_test
test function
[root@node1 data]# 

```

## 查看系统定义的函数

```
[root@node1 data]# declare -F|tail -5
declare -f command_not_found_handle
declare -f dequote
declare -f func_test
declare -f quote
declare -f quote_readline

```

## 删除函数

```
[root@node1 data]# unset func_test
[root@node1 data]# func_test
bash: func_test: command not found...
[root@node1 data]# 

```

## 查看系统版本

```
[root@node1 data]# ./functi_test.sh 
OS version is 7
[root@node1 data]# 
[root@node1 data]# 
[root@node1 data]# cat functi_test.sh 
#!/bin/bash
#
func_os_version () {
    sed -nr 's/.* ([0-9]+)\..*/\1/p' /etc/redhat-release
}
echo OS version is `func_os_version`
[root@node1 data]# 

```

## source执行函数

```
[root@node1 data]# cat functions.sh 
#!/bin/bash
source functions
func_os_version
[root@node1 data]# cat functions
func_os_version () {
    sed -nr 's/.* ([0-9]+)\..*/\1/p' /etc/redhat-release
}
[root@node1 data]# ./functions.sh 
7
[root@node1 data]# 

```

## 考试成绩查询

```
[root@node1 data]# cat functions
func_os_version () {
    sed -nr 's/.* ([0-9]+)\..*/\1/p' /etc/redhat-release
}
func_is_digit () {
    if [ ! "$1" ];then
        echo Usage:func_is_digit number
        return 10
    elif [[ $1 =~ ^[[:digit:]]+$ ]];then
        return 0
    else
        echo "Not a digit"
        return 1
    fi    
}
[root@node1 data]# cat sorce.sh 
#!/bin/bash
#
source functions
read -p "Input your score:" SCORE
func_is_digit $SCORE
if [ $? -ne 0 ];then
    exit
else
    if [ $SCORE -lt 60 ];then
        echo "Your are loser"
    elif [ $SCORE -lt 80 ];then
        echo "soso"
    else
        echo "very good"
    fi
fi
[root@node1 data]# 

```

## 引用系统函数

```
[root@node1 init.d]# . /etc/init.d/functions 
[root@node1 init.d]# action "Are you OK?"
Are you OK?                                                [  OK  ]
[root@node1 init.d]# action "Are you OK?" false
Are you OK?                                                [FAILED]
[root@node1 init.d]# 



```

## 函数影响本地变量的值

```
[root@node1 data]# num=200; func_test () { num=100;echo $num; }
[root@node1 data]# func_test
100
[root@node1 data]# echo $num
100

```

## 本地变量

```
[root@node1 data]# num=200; func_test () { local num=100;echo $num; }
[root@node1 data]# func_test
100
[root@node1 data]# echo $num
200

```

```
[root@node1 data]# { echo x; echo y; }
x
y

```

## 递归消耗内存

```
[root@node1 data]# cat recursion.sh 
#!/bin/bash
#
echo $BASHPID
sleep 1
./recursion.sh

```

## 堆栈的消耗内存

```
[root@node1 ~]# func_test () { let i++; echo $i;func_test; }
[root@node1 ~]# func_test

```

## 阶乘脚本

```
[root@node1 data]# cat ./fact.sh 
#!/bin/bash
#
fact () {
    if [ $1 -eq 0 ];then
        echo 1
    else
        echo $[`fact $[$1-1]`*$1]
    fi
}
fact $1
[root@node1 data]# ./fact.sh 3
6

```

## fork炸弹

```
[root@node1 data]# :(){ :|:& };:
[1] 39114
[root@node1 data]# free 

```

## trap 信号

```
[root@node1 ~]# cat trap.sh 
#!/bin/bash
#
trap 'echo press ctrl+c' int
trap -p
for ((i=0;i<10;i++));do
    echo $i
    sleep 1
done

```

## 忽略信号

```
[root@node1 ~]# cat trap.sh
#!/bin/bash
#
trap 'echo press ctrl+c' int
trap -p
for ((i=0;i<10;i++));do
    echo $i
    sleep 1
done

trap '' 2
trap -p
for ((i=10;i<20;i++));do
    echo $i
    sleep 1
done

```

## 恢复信号

```
[root@node1 ~]# cat trap.sh
#!/bin/bash
#
trap 'echo press ctrl+c' int
trap -p
for ((i=0;i<10;i++));do
    echo $i
    sleep 1
done

trap '' 2
trap -p
for ((i=10;i<20;i++));do
    echo $i
    sleep 1
done

trap '-' 2
trap -p
for ((i=20;i<30;i++));do
    echo $i
    sleep 1
done

```

## 执行一半收尾工作

```
[root@node1 ~]# cat trap2.sh
#!/bin/bash
finish () {
    echo finish is excuted
}
trap finish exit
for ((i=0;i<5;i++));do
    echo $i
    sleep 1
done

```

## 数组

```
[root@node1 ~]# TITLE[0]=CEO
[root@node1 ~]# TITLE[1]=COO
[root@node1 ~]# TITLE[2]=CTO
[root@node1 ~]# echo ${TITLE[0]}
CEO
[root@node1 ~]# 
[root@node1 ~]# echo ${TITLE[1]}
COO
[root@node1 ~]# echo ${TITLE[2]}
CTO
[root@node1 ~]# echo $TITLE
CEO
[root@node1 ~]# echo ${TILE[@]}

[root@node1 ~]# echo ${TITLE[@]}
CEO COO CTO
[root@node1 ~]# echo ${TITLE[@]}
CEO COO CTO
[root@node1 ~]# TITLE[${#TITLE[@]}]=xidu
[root@node1 ~]# echo ${#TITLE[@]}
4
[root@node1 ~]# 
[root@node1 ~]# NAME=(mage zhang wang)
[root@node1 ~]# echo ${NAME[1]}
zhang
[root@node1 ~]# 
[root@node1 ~]# echo ${NAME[0]}
mage
[root@node1 ~]# echo ${NAME[2]}
wang


```

## 列表的数组

```
[root@node1 ~]# number=({1..10})
[root@node1 ~]# echo ${number[@]}
1 2 3 4 5 6 7 8 9 10
[root@node1 ~]# 

```

## 文件列表数组

```
[root@node1 ~]# file=(*)
[root@node1 ~]# echo ${file[0]}
anaconda-ks.cfg
[root@node1 ~]# echo ${file[1]}
initial-setup-ks.cfg
[root@node1 ~]# echo ${file[5]}
tt

```

```
[root@node1 ~]# student=([0]=xiaoming [2]=xiaohong [4]=tom)
[root@node1 ~]# echo ${student[1]}

[root@node1 ~]# echo ${student[2]}
xiaohong
[root@node1 ~]# echo ${student[@]}
xiaoming xiaohong tom
[root@node1 ~]# 

```

## read -a 数组

```
[root@node1 ~]# read -a CAR
benz BMW AUDI 
[root@node1 ~]# echo ${CAR[1]}
BMW
[root@node1 ~]# echo ${CAR[0]}
benz
[root@node1 ~]# echo ${CAR[2]}
AUDI
[root@node1 ~]# echo ${CAR[*]}
benz BMW AUDI

```

## 关联数组声明

```
[root@node1 ~]# MENU=([gongbaojiding]=20 [baoyu]=100 [haishen]=50)
[root@node1 ~]# echo ${MENU[gongbaojiding]}
50
[root@node1 ~]# echo ${MENU[baoyu]}
50
[root@node1 ~]# declare -A MENU
-bash: declare: MENU: cannot convert indexed to associative array
[root@node1 ~]# unset MENU
[root@node1 ~]# declare -A MENU
[root@node1 ~]# MENU=([gongbaojiding]=20 [baoyu]=100 [haishen]=50)
[root@node1 ~]# echo ${MENU[baoyu]}
100
[root@node1 ~]# echo ${MENU[gongbaojiding]}
20

```

## 数组报警脚本

```
[root@node1 ~]# cat array_diskcheck.sh 
#!/bin/bash
#
declare -A DISK
WARNING=3
df |grep '^/dev/sd' > df.log
while read line; do
    INDEX=`echo $line |sed -rn 's#^([^[:space:]]+) .*#\1#p'`
    DISK[$INDEX]=`echo $line |sed -rn 's#^.* ([0-9]+)%.*#\1#p'`
    if [ ${DISK[$INDEX]} -gt $WARNING ];then
        wall $INDEX: ${DISK[$INDEX]}
    fi
done < df.log

```

## 数组的切片

```
[root@node1 ~]# NUM=({1..10})
[root@node1 ~]# echo ${NUM[@]}
1 2 3 4 5 6 7 8 9 10
[root@node1 ~]# echo ${NUM[@]:2}
3 4 5 6 7 8 9 10
[root@node1 ~]# echo ${NUM[@]:2:3}
3 4 5
[root@node1 ~]# 

```

## 最大最小值

```
[root@node1 ~]# bash ./array_max_min.sh
ALL random are 2738 25347 26614 1408 11309 31797 13482 15463 7360 3241
max=31797 min=2738
[root@node1 ~]# cat ./array_max_min.sh
#!/bin/bash
#
declare -a RAND
for ((i=0;i<10;i++));do
    RAND[$i]=$RANDOM
    if [ $i -eq 0 ];then
        MAX=${RAND[$i]}
        MIN=$MAX
    else
        [ ${RAND[$i]} -gt $MAX ]   && { MAX=${RAND[$i]};continue; }
        [ ${RAND[$i]} -lt $MIN ]   && MIX=${RAND[$i]}
    fi
done
echo ALL random are ${RAND[*]}
echo max=$MAX min=$MIN

```

## 字符串切片

```
[root@node1 ~]# NAME=magedu
[root@node1 ~]# echo ${#NAME}
6
[root@node1 ~]# 

```

```
[root@node1 ~]# NAME=mage教育
[root@node1 ~]# echo ${#NAME}
6

```

```
[root@node1 ~]# NAME=mage教育
[root@node1 ~]# echo ${#NAME}
6
[root@node1 ~]# echo ${NAME:2}
ge教育
[root@node1 ~]# echo ${NAME:2:2}
ge
[root@node1 ~]# echo ${NAME:4:2}
教育
[root@node1 ~]# echo ${NAME: -2}
教育

[root@node1 ~]# echo ${NAME:2:-2}
ge
[root@node1 ~]# echo ${NAME: -4:-2}
ge

```

```
[root@node1 ~]# echo $LINE
abcroot:x:0:0:root:/root:/bin/bash
[root@node1 ~]# echo ${LINE#*root}
:x:0:0:root:/root:/bin/bash
```

## 贪婪模式删除字符串

```
[root@node1 ~]# echo ${LINE##*root}
:/bin/bash
[root@node1 ~]# 

```

## 后往前删除

```
[root@node1 ~]# echo ${LINE%root*}
abcroot:x:0:0:root:/
[root@node1 ~]# echo ${LINE%%root*}
abc
[root@node1 ~]# 

```

## 变量赋值

```
[root@node1 ~]# name=${title-mage}
[root@node1 ~]# set |grep title
[root@node1 ~]# echo $name
mage
[root@node1 ~]# title=""
[root@node1 ~]# name=${title-mage}
[root@node1 ~]# echo $name

[root@node1 ~]# title="ceo"
[root@node1 ~]# name=${title-name}
[root@node1 ~]# echo $name
ceo

```

```
[root@node1 ~]# declare -l name=CEO
[root@node1 ~]# echo $name
ceo
[root@node1 ~]# declare -u name=ceo
[root@node1 ~]# echo $name
CEO

```

## eval命令

```
[root@node1 ~]# type eval
eval is a shell builtin
[root@node1 ~]# n=10;echo {1..$n}
{1..10}
[root@node1 ~]# n=10;eval echo {1..$n}
1 2 3 4 5 6 7 8 9 10
[root@node1 ~]#  
[root@node1 ~]# for i in `eval echo {1..$n}`;do echo $i;done
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
[root@node1 ~]# CMD=hostname

[root@node1 ~]# $CMD
node1
[root@node1 ~]# CMD=pwd
[root@node1 ~]# $CMD
/root


```

```
[root@node1 ~]# title=name
[root@node1 ~]# name=mage
[root@node1 ~]# echo $title
name
[root@node1 ~]# echo \$$title
$name
[root@node1 ~]# eval echo 

[root@node1 ~]# eval echo \$$title
mage
[root@node1 ~]# echo ${!title}
mage

```

## 创建临时目录

```
[root@node1 ~]# mktemp -d file`date +'%F_%T'`XXXX
file2020-10-16_11:22:01d1X2

[root@node1 ~]# mktemp -d -p /tmp file`date +'%F_%T'`XXXX
/tmp/file2020-10-16_11:47:086X6Z


```

## install 命令

```
[root@node1 ~]# install /etc/issue .
[root@node1 ~]# ls
issue
[root@node1 ~]# install /etc/fstab .
[root@node1 ~]# ls
fstab  issue
[root@node1 ~]# install /etc/fstab fastab.bak -m 600 -o shaoxh 
[root@node1 ~]# ll
total 12
-rw-------. 1 shaoxh root 477 Oct 16 11:49 fastab.bak
-rwxr-xr-x. 1 root   root 477 Oct 16 11:48 fstab
-rwxr-xr-x. 1 root   root  23 Oct 16 11:48 issue
[root@node1 ~]# 

```

## expect 命令

```
[root@node1 ~]#  rpm -ql expect
/usr/bin/autoexpect
/usr/bin/dislocate
/usr/bin/expect
/usr/bin/ftp-rfc
/usr/bin/kibitz
/usr/bin/lpunlock
/usr/bin/mkpasswd
/usr/bin/passmass
/usr/bin/rftp
/usr/bin/rlogin-cwd
/usr/bin/timed-read
/usr/bin/timed-run
/usr/bin/unbuffer
/usr/bin/weather
/usr/bin/xkibitz
/usr/lib64/libexpect.so
/usr/lib64/libexpect5.45.so
/usr/lib64/tcl8.5/expect5.45
/usr/lib64/tcl8.5/expect5.45/pkgIndex.tcl
/usr/share/doc/expect-5.45
/usr/share/doc/expect-5.45/FAQ
/usr/share/doc/expect-5.45/HISTORY
/usr/share/doc/expect-5.45/NEWS
/usr/share/doc/expect-5.45/README
/usr/share/man/man1/autoexpect.1.gz
/usr/share/man/man1/dislocate.1.gz
/usr/share/man/man1/expect.1.gz
/usr/share/man/man1/kibitz.1.gz
/usr/share/man/man1/mkpasswd.1.gz
/usr/share/man/man1/passmass.1.gz
/usr/share/man/man1/tknewsbiff.1.gz
/usr/share/man/man1/unbuffer.1.gz
/usr/share/man/man1/xkibitz.1.gz
[root@node1 ~]# /usr/bin/mkpasswd 
zaVr69#Kv
[root@node1 ~]# /usr/bin/mkpasswd -l 20 -d 10
j(yy8P98Ym4771g483nn

```

## 捕获

```
[root@node1 ~]# expect 
expect1.1> expect "hello" {send "you said hello"}
aaa
bbb
hello
you said helloexpect1.2> expect 'hi' {send "hi} \
+> 'hehe' {send 'hehe'} \
+> 'bye' {send 
+> 'bye'}
bye
expect1.3> 

```

## 单一分支模式语法

```
[root@node1 ~]# expect 
expect1.1> expect "hi" {send "you said hi\n"}
aaa
hi
you said hi
expect1.2> 

```

## 多分支模式语法

```
[root@node1 ~]# expect 
expect1.1> expect "hi" {send "you said hi\n"} \
+> "hehe" {send "hehe yourself\n"} \
+> "bye" {send "good bye\n"}
bye
good bye
expect1.2> 

```

```
[root@node1 ~]# expect 
expect1.1> expect {  
+> "hi" {send "you said hi\n"}
+> "hehe" {send "hehe yourself\n"}
+> "bye" {send "good bye\n"}
+> }
hehe
hehe yourself
expect1.2> 

```

## expect脚本

```
[root@node1 ~]# cat expect 
#!/usr/bin/expect
#
spawn scp /root/sum.sh 192.168.1.56:/root
expect {
       "yes/no" {send "yes\n";exp_continue}
       "password" {send "shaoxh\n"}
}
expect eof

```

## 登录其他主机

```
[root@node1 ~]# !rm
rm -rf /root/.ssh/
[root@node1 ~]# cat expect2 
#!/usr/bin/expect
spawn ssh 192.168.1.56
expect {
    "yes/no" {send "yes\n";exp_continue}
    "password" { send "shaoxh\n"}
}
interact
[root@node1 ~]# expect expect2 
spawn ssh 192.168.1.56
The authenticity of host '192.168.1.56 (192.168.1.56)' can't be established.
ECDSA key fingerprint is SHA256:zpZvV9s9dFXUlE86N8SSqruRXqPu5cARuNtEX7v2dg8.
ECDSA key fingerprint is MD5:44:6b:2d:c0:f2:d8:a1:d1:48:30:34:5a:2b:79:f1:e1.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.56' (ECDSA) to the list of known hosts.
root@192.168.1.56's password: 
Last login: Sat Oct 17 15:43:10 2020 from 192.168.1.47
[root@node2 ~]# 

```

## expect变量

```
[root@node1 ~]# !rm
rm -rf /root/.ssh/
[root@node1 ~]# cat expect3
#!/usr/bin/expect
set ip 192.168.1.56
set user root
set password shaoxh
set timeout 10
spawn ssh $user@$ip
expect {
    "yes/no" {send "yes\n";exp_continue}
    "password" {send "$password\n"}
}
interact
[root@node1 ~]# expect expect3
spawn ssh root@192.168.1.56
The authenticity of host '192.168.1.56 (192.168.1.56)' can't be established.
ECDSA key fingerprint is SHA256:zpZvV9s9dFXUlE86N8SSqruRXqPu5cARuNtEX7v2dg8.
ECDSA key fingerprint is MD5:44:6b:2d:c0:f2:d8:a1:d1:48:30:34:5a:2b:79:f1:e1.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.56' (ECDSA) to the list of known hosts.
root@192.168.1.56's password: 
Last login: Sat Oct 17 15:47:51 2020 from 192.168.1.47
[root@node2 ~]# 

```

## 位置参数

```
[root@node1 ~]# vim expect4
[root@node1 ~]# 
[root@node1 ~]# expect expect4 192.168.1.56 root shaoxh
spawn ssh root@192.168.1.56
root@192.168.1.56's password: 
Last login: Sat Oct 17 15:48:34 2020 from 192.168.1.47
[root@node2 ~]# exit
logout
Connection to 192.168.1.56 closed.
[root@node1 ~]# cat expect4 
#!/usr/bin/expect
#
set ip [lindex $argv 0]
set user [lindex $argv 1]
set password [lindex $argv 2]
spawn ssh $user@$ip
expect {
    "yes/no" { send "yes\n";exp_continue}
    "password" {send "$password\n"}
}
interact

```

## 执行多个命令

```
[root@node1 ~]# vim expect5
[root@node1 ~]# 
[root@node1 ~]# expect expect5 192.168.1.56 root shaoxh
spawn ssh root@192.168.1.56
root@192.168.1.56's password: 
Last login: Sat Oct 17 15:54:46 2020 from 192.168.1.47
[root@node2 ~]# useradd haha
[root@node2 ~]# echo shaoxh|passwd --stdin haha
Changing password for user haha.
passwd: all authentication tokens updated successfully.
[root@node2 ~]# exit
logout
Connection to 192.168.1.56 closed.
[root@node1 ~]# cat expect5 
#!/usr/bin/expect 
#
set ip [lindex $argv 0]
set user [lindex $argv 1]
set password [lindex $argv 2]
set timeout 10
spawn ssh $user@$ip
expect {
    "yes/no" {send "yes\n";exp_continue}
    "password" {send "shaoxh\n"}
}
expect "]#" {send "useradd haha\n"}
expect "]#" {send "echo shaoxh|passwd --stdin haha\n"}
send "exit\n"
expect eof

```

## shell中调用expect

```
root@node1 ~]# bash expect6.sh 192.168.1.56 root shaoxh
spawn ssh root@192.168.1.56
root@192.168.1.56's password: 
Last login: Sat Oct 17 16:42:46 2020 from 192.168.1.47
[root@node2 ~]# useradd hehe
[root@node2 ~]# echo shaoxh|passwd --stdin hehe
Changing password for user hehe.
passwd: all authentication tokens updated successfully.
[root@node2 ~]# exit
logout
Connection to 192.168.1.56 closed.
[root@node1 ~]# cat expect6.sh 
#!/bin/bash
#
ip=$1
user=$2
password=$3
expect <<EOF
set timeout 20
spawn ssh $user@$ip
expect {
    "yes/no" {send "yes\n";exp_continue}
    "password" {send "$password\n"}
}
expect "]#" {send "useradd hehe\n"}
expect "]#" {send "echo shaoxh|passwd --stdin hehe\n"}
expect "]#" {send "exit\n"}
expect eof
EOF
[root@node1 ~]# 

```

## 多台主机

```
[root@node1 ~]# cat hostlist.txt 
192.168.1.56
[root@node1 ~]# cat expect7.sh 
#!/bin/bash
#
while read ip;do
user=root
password=shaoxh
expect <<EOF
set timeout 20
spawn ssh $user@$ip
expect {
    "yes/no" {send "yes\n";exp_continue}
    "password" {send "$password\n"}
}
expect "]#" {send "useradd meimei\n"}
expect "]#" {send "echo shaoxh|passwd --stdin meimei\n"}
expect "]#" {send "exit\n"}
expect eof
EOF
done < hostlist.txt
[root@node1 ~]# 

```

