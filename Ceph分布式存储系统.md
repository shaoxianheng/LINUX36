# Ceph分布式存储系统

## 1.1  存储设备

```
DAS：IDE、SATA、SCSI、SAS、USB
NAS：NFS、CIFS
SAN：SCSI、FC SAN、iSCSI
```

EMC、NetAPP、IBM

## 1.2 下载 ceph仓库

```
node1
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/ceph/rpm-mimic/el7/noarch/ceph-release-1-1.el7.noarch.rpm
yum -y install epel-release
```

## 1.3 关闭防火墙selinux 时间同步，主机名

```
[cephadm@ceph-admin ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.43.111 mon01.ilinux.io mon01 stor01.ilinux.io stor01
192.168.43.112 mon02.ilinux.io mon02 stor02.ilinux.io stor02
192.168.43.113 mon03.ilinux.io mon04 stor03.ilinux.io stor03
192.168.43.114 stor04.ilinux.io stor04

```

添加硬盘b c

```
 ansible all -m shell -a 'echo "- - -" > /sys/class/scsi_host/host0/scan'
```



## 配置ansible

```
[root@001 ~]# cat > /etc/ansible/hosts
[all]
192.168.120.78
192.168.120.117
192.168.120.127
192.168.120.120
192.168.120.71
ctrl +c结束
```

## 添加用户

```
ansible all -m shell -a 'useradd cephadm'
ansible all -m shell -a 'echo cephadm:shaoxh|chpasswd'

```

授权无密码运行sudo

```
ansible all -m shell -a 'echo cephadm "ALL=(root)" NOPASSWD:ALL > /etc/sudoers.d/cephadm'
ansible all -m shell -a 'chmod 0440 /etc/sudoers.d/cephadm'
```

安装ceph-deploy

```
yum update
yum -y install ceph-deploy python-setuptools python2-subprocess32
```

初始化RADOS 集群

```
切换cephadm用户
 su - cephadm
 mkdir ceph-cluster
```

```
                           192.168.43.233     001   192.168.190.128   
 public-network            192.168.43.142     002   192.168.190.129   cluster-network
                           192.168.43.109     003   192.168.190.130
                           192.168.43.241     004   192.168.190.133
                           192.168.43.110     005   192.168.190.132
```

```
                                                172.18.199.71   mon01
                                                172.18.199.72   mon02
                                                           73   mon03
                                                           74   stor04
```

## 在Ceph各节点创建新用户

首先需要在各节点一管理员的身份创建一个专用于ceph-deploy的特定用户账号，例如cephadm（建议不要使用ceph），并为其设置认证密码（例如shaoxh):

```
[root@ceph-admin ~]# useradd cephadm
[root@ceph-admin ~]# echo cephadm:shaoxh|chpasswd
```

而后，确保这些节点上新创建的用户cephadm都有无密钥运行sudo命令的权限。

```
[root@ceph-admin ~]# cat /etc/sudoers.d/cephadm
cephadm         ALL=(root)   NOPASSWD: ALL
[root@ceph-admin ~]# su - cephadm
[cephadm@ceph-admin ~]$ sudo -l
....
 (root) NOPASSWD: ALL

[root@ceph-admin ~]# scp -rp /etc/sudoers.d/cephadm stor01:/etc/sudoers.d/   //拷贝至（stor01-stor04)
```

## cephadm生成密钥

使用ssh-keygen 生成密钥后拷贝至其他主机，使其用一个密钥来验证，

```
[cephadm@ceph-admin ~]$ ssh-keygen -P ''
[cephadm@ceph-admin ~]$ ssh-copy-id -i .ssh/id_rsa.pub cephadm@localhost
[cephadm@ceph-admin ~]$ scp -rp .ssh/ cephadm@stor02:/home/cephadm/     //拷贝至（stor01-stor04)
```

## 在管理节点安装ceph-deploy

Ceph存储集群的部署过的过程可通过管理节点使用ceph-deploy全程进行，这里首先在管理节点安装ceph-deploy及其依赖到的程序包：

```
[root@ceph-admin ~]# yum -y update
[root@ceph-admin ~]# yum -y install ceph-deploy python-setuptools python2-subprocess32
```

## 部署RADOS存储集群

### 初始化RADOS集群

1.首先在管理节点上以cephadm用户创建集群相关的配置文件目录：

```
[cephadm@ceph-admin ~]$ mkdir ceph-cluster
[cephadm@ceph-admin ~]$ cd ceph-cluster
```

2.初始化第一个MON节点，准备创建集群：

初始化第一个MON节点的命令格式为“ceph-deploy new {initial-monitor-node(s)}", 本示例中，stor01即为第一个MON节点名称，其名称必须与节点当前使用使用的主机名称保持一致。运行如下命令即可生成初始化配置：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy new --cluster-network 192.168.190.0/24 --public-network 192.168.43.0/24 mon01.ilinux.io
```

3.编辑生成ceph.conf配置文件，在[global]配置段中设置Ceph集群面向客户端通信时使用的IP地址所在的网路，即公网网路地址：

public_network = 192.168.43.0/24

4.安装Ceph集群

ceph-deploy命令能够以远程的方式连入Ceph集群各节点完成程序包安装等操作，命令格式如下：

   cehp-deploy  install {ceph-node} [{ceph-node}...]

因此，若要将stor01 、stor02和stor03 配置为Ceph 集群节点，则执行如下命令即可：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy install stor01 stor02 stor03 stor04

```

提示：若需要在集群各节点独立安装ceph程序包，其方法如下：

```
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/ceph/rpm-mimic/el7/noarch/ceph-release-1-1.el7.noarch.rpm
yum -y install epel-release
yum -y install ceph ceph-radosgw
```

本实验的步骤是

```
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/ceph/rpm-mimic/el7/noarch/ceph-release-1-1.el7.noarch.rpm
yum -y install epel-release
yum -y install ceph ceph-radosgw
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy install --no-adjust-repos mon01 mon02 mon03 stor04
```

5.配置初始MON节点，并收集所有密码：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy mon create-initial

```

6.把配置文件和admin密钥拷贝Ceph集群各节点，以免得每次执行”ceph“命令时不得不明确指定MON节点地址和ceph.client.admin.keyring:

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy admin mon01 mon02 mon03 stor04
```

而后在Ceph集群中需要运行ceph命令的节点上（或所有节点）以root用户身份设定用户cephadm能够读取/etc/ceph/ceph.client.admin.keyring文件:

```
[root@mon01 ceph]# setfacl -m u:cephadm:rw /etc/ceph/ceph.client.admin.keyring

```

7.配置Manager节点，启动ceph-mgr进程 （仅Luminious+版本）:

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy mgr create mon03
```

安装一个客户端的ceph-common包

```
[root@ceph-admin ~]# yum -y install ceph-common
[cephadm@ceph-admin ceph-cluster]$ ceph -s
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy admin ceph-admin
```

## 向RADOS集群添加OSD

### 列出并擦净磁盘

”ceph-deploy disk "命令可以检查并列出OSD节点所有可用的磁盘的相关信息：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy disk list mon01 mon02
```

而后，在管理节点上使用ceph-deploy命令擦除计划专用于OSD磁盘上的所有分区表和数据以便用于OSD，命令格式为“ceph-deploy disk zap {osd-server-name} {disk-name}" ,需要注意的是此步会清除目标设备上的所有数据。下面分别擦净mon01上用于OSD的磁盘设备：（mon01，mon02,mon03，stor04)

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy disk zap mon01 /dev/sdb
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy disk zap mon01 /dev/sdc
```

提示：若设备上此前有数据，则可能需要在相应节点上以root用户使用”ceph-volume lvm zap --destroy {DEVICE}" 命令进行；

### 添加OSD

早期版本的ceph-deploy命令支持在将添加OSD的过程分为两个步骤：准备OSD和激活OSD，但新版中，此种操作方式已经被废除，添加OSD的步骤只能由命令

“ceph-deploy osd create {node} --data {data-disk}" 一次完成，，默认使用的存储引擎为bluestore。

如下命令即可分别把mon01、mon02、mon03和stor04上的设备sdb、sdc添加为OSD：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy osd create --data /dev/sdb mon01
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy osd create --data /dev/sdb mon02
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy osd create --data /dev/sdb mon03
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy osd create --data /dev/sdb stor04
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy osd create --data /dev/sdc mon01
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy osd create --data /dev/sdc mon02
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy osd create --data /dev/sdc mon03
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy osd create --data /dev/sdc stor04
[cephadm@ceph-admin ceph-cluster]$ ceph -s

```

然后可使用”ceph-deploy osd list" 命令列出指定节点上的OSD：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy osd list mon01 mon02
```

事实上，管理员也可用使用ceph命令查看OSD的相关信息：

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd stat
8 osds: 8 up, 8 in; epoch: e33
```

或者使用如下命令了解相关的信息：

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd dump
[cephadm@ceph-admin ceph-cluster]$ ceph osd ls
```

### 从RADOS集群中移除OSD的方法

Ceph集群中的一个OSD通常应于一个设备，且运行于专用的守护进程。在某OSD设备出现故障，或管理员出于管理之需确实要移除特定OSD设备时，需要先停止相关的守护进程，而后再进行移除操作。对于Luminous及其之后的版本来说，停止和移除命令格式分别如下所示：

  1.停用设备：ceph osd out {osd-num}

  2.停止进程：sudo systemctl stop ceph-osd@{osd-num}

  3.移除设备： ceph osd purge {id} --yes-i-really-mean-it

若类似如下的OSD的配置信息存在于ceph.conf配置文件中,管理员再删除OSD之后手动将其删除。

[osd.1]

​			host = {hostname}

不过，对于Luminous之前的版本来说，管理员需要依次手动执行如下步骤删除OSD设备：

1. 于CRUSH运行图中移除设备：ceph osd crush remove {name}

2. 移除OSD的认证key： ceph auth del osd.{osd-num}

3. 最后移除OSD设备：ceph osd  rm {osd-num}

   

### 测试上传/下载数据对象

存取数据时，客户端必须首先连接至RADOS集群上某存储池，而后根据对象名称由相关的CRUSH规则完成数据对象寻址。于是，为了测试集群的数据存取功能，这里首先创建一个用于测试的存储池mypool，并设定其PG数量为32个。

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool create mypool 32 32
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool ls
[cephadm@ceph-admin ceph-cluster]$ rados lspools
```

而后即可将测试文件上传至存储池中，例如下面的“rados put” 命令将 jpg 文件上传至mypool存储池，对象名称依然保留文件名，而“rados ls” 命令则可以列出指定存储池中的数据对象。

```
[cephadm@ceph-admin ceph-cluster]$ rados put day.jpg /usr/share/backgrounds/day.jpg -p mypool
[cephadm@ceph-admin ceph-cluster]$ rados ls -p mypool
day.jpg
```

而“ceph osd map” 命令可以获取到存储池中数据对象的具体位置信息：

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd map mypool day.jpg
```

删除数据对象，“rados rm” 命令时较为常用的一种方式：

```
[cephadm@ceph-admin ceph-cluster]$ rados rm day.jpg -p mypool
```

删除存储池命令存在数据丢失的风险，Ceph于是默认禁止此类操作。管理员需要再ceph.conf配置文件中启用支持删除存储池的操作后，放可使用类似如下命令删除存储池。

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool rm mypool mypool --yes-i-really-really-mean-it

```

## 扩展Ceph集群

### 扩展监视器节点

Ceph存储集群至少运行一个Ceph Monitor和一个Ceph Manager， 生产环境中，为了实现高可用性，Ceph存储集群通常运行多个监视器，以免单监视器整个存储集群崩溃。Ceph使用Paxos算法，该算法需要半数以上的监视器（大于n/2,其中n为总监视器数量）才能形成法定人数。尽管此非必须，但奇数个监视器往往更好。

“ceph-deploy mon add {ceph-nodes}" 命令可以一次添加一个监视节点到集群中。例如，下面的命令可以将集群中的stor02和stor03也运行为监视器节点：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy mon add mon02
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy mon add mon03
```

设置完成后，可以在ceph客户端上查看监视器及法定人数的相关状态：

```
[cephadm@ceph-admin ceph-cluster]$ ceph quorum_status --format json-pretty
```

### 扩展Manager 节点

Ceph Manager守护进程以”Active/Standby“ 模式运行，部署其它ceph-mgr守护程序可确保在Active节点或其上的ceph-mgr守护进程故障时，其中的一个Standby实例可以在不中断服务器的情况下接管其任务。

”ceph-deploy mgr create {new-manager-nodes}" 命令可以一次添加多个Manager节点。下面的命令可以将stor04节点作为备用的Manager运行：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy mgr create stor04
```

## Ceph存储集群的访问接口

### Ceph块设备接口（RBD）

Ceph块设备，也称为RADOS块这部（简称RBD），是一种基于RADOS存储系统支持超配（thin-provisioned)、可伸缩的条带化数据存储系统，它通过librbd库与OSD进行交换。RBD为KVM等虚拟化技术和云OS（如OpenStack和CloudStack）提供高性能和无线扩展性的存储后端，这些系统依赖于libvirt和QEM实用程序于RBD进行集成。

客户端基于librbd库即可将RADOS存储集群用作块设备，不过，用于rbd的存储池也需要事先启用rbd功能并进行初始化。例如，下面的命令创建一个为rbddata的存储池，在启动rdb功能后对其进行初始化：

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool create rbdpool  64 64
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool application enable rbdpool rbd
[cephadm@ceph-admin ceph-cluster]$ rbd pool init -p rbdpool
```

不过，rbd存储池并不能直接用于块设备，而是需要事先在其中按需创建映像（image），并把映像文件作为块设备使用。rbd命令可用于创建、查看及删除块设备相在的映像（image），以及克隆映像、创建快照、将映像回滚到快照和查看快照等管理操作。例如下面的命令能够创建一个名为myimg的映像：

```
[cephadm@ceph-admin ceph-cluster]$ rbd create --size 2G rbdpool/myimg
[cephadm@ceph-admin ceph-cluster]$ rbd ls -p rbdpool
myimg
```

映像的相关信息则可以使用“rbd info” 命令获取：

```
[cephadm@ceph-admin ceph-cluster]$ rbd info rbdpool/myimg
```

在客户端主机上，用户通过内核级的rbd驱动识别相关的设备，即可对其进行区分、创建文件系统并挂载使用。

### 启用radosgw接口

RGW并非必须的接口，仅在需要用到于S3和Swift兼容的RESTFul接口时才需要部署RGW实例，相关的命令为”ceph-deploy rgw create {gateway-node}" 。例如，下面的命令用于把mon01部署为rgw主机：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy rgw create mon01
浏览器访问：http://192.168.0.111:7480/       
<ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<Owner>
<ID>anonymous</ID>
<DisplayName/>
</Owner>
<Buckets/>
</ListAllMyBucketsResult>
```

添加完成后，“ceph -s " 命令的services 一段中会输出相关信息：

```
[cephadm@ceph-admin ceph-cluster]$ ceph -s
 cluster:
    id:     903285d9-d8c2-478b-85cb-70ca536eb33a
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum mon01,mon02,mon03
    mgr: mon03(active), standbys: stor04
    osd: 8 osds: 8 up, 8 in
    rgw: 1 daemon active

  data:
    pools:   6 pools, 128 pgs
    objects: 192  objects, 1.3 KiB
    usage:   8.1 GiB used, 44 GiB / 52 GiB avail
    pgs:     128 active+clean
```

默认情况下，RGW实例监听于TCP协议的7480端口，需要算定时，可以通过在运行RGW的节点上编辑器其主配置文件ceph.conf进行修改，相关参数如下所示：

```
[client]
rgw_frontends = "civetweb port =8080"
```

而后需要重启相关的服务，命令格式为“systemctl restart ceph-radosgw@rgw.{node-name}",例如重启mon01上的RGW，可以以root用户运行如下命令：

```
[root@mon01 ~]# systemctl restart ceph-radosgw@rgw.mon01
```

RGW会在rados集群上生成包括如下存储池的一系列存储池:

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool ls
mypool
rbdpool
.rgw.root
default.rgw.control
default.rgw.meta
default.rgw.log
```

RGW 提供的时REST接口，客户端通过http与其进行交互，完成数据的增删改查等管理操作。

### 启用文件系统（CephFS）接口

CephFS 需要至少运行一个元数据服务器（MDS）守护进程（ceph-mds），此进程管理与Ceph上存储的文件相关的元数据，并协调对Ceph存储集群的访问。因此，若要使用Ceph接口，需要在存储集群中至少部署一个MDS实例。”ceph-deploy mds create {ceph-node}“ 命令可以完成此功能。例如，在mon02上启用MDS：

```
[cephadm@ceph-admin ceph-cluster]$ ceph-deploy mds create mon02
```

查看MDS的相关状态可以发现，刚添加的MDS处于Standby模式：

```
[cephadm@ceph-admin ceph-cluster]$ ceph mds stat
, 1 up:standby
```

使用CephFS 之前需要事先于集群中创建一个文件系统，并为其分别指定元数据和数据相关的存储池。下面创建一个名为ceph的文件系统用于测试，它使用cephfs-metadata为元数据存储池，使用cephfs-data为数据存储池：

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool create cephfs-metadata 32 32
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool create cephfs-data 64 64
[cephadm@ceph-admin ceph-cluster]$ ceph fs new cephfs cephfs-metadata cephfs-data
```

而后即可使用如下命令”ceph fs status <fsname>" 查看文件系统的相关状态，例如：

```
[cephadm@ceph-admin ceph-cluster]$ ceph fs status cephfs
```

此时，MDS的状态已经发生了改变：

```
[cephadm@ceph-admin ceph-cluster]$ ceph mds stat
cephfs-1/1/1 up  {0=mon02=up:active}
```

随后，客户端通过内核中的cephfs文件系统接口即可挂载使用cephfs文件系统，或者通过FUSE接口于文件系统进行交互。

查看ceph实时状态

```
[cephadm@ceph-admin ceph-cluster]$ ceph pg stat
224 pgs: 224 active+clean; 3.6 KiB data, 8.1 GiB used, 44 GiB / 52 GiB avail
```

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool stats cephfs-data
pool cephfs-data id 8
  nothing is going on
```

```
[cephadm@ceph-admin ceph-cluster]$ ceph df
```

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd stat
8 osds: 8 up, 8 in; epoch: e70
```

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd dump
```

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd tree
```

```
[cephadm@ceph-admin ceph-cluster]$ ceph mon stat
[cephadm@ceph-admin ceph-cluster]$ ceph mon dump
```

```
[root@mon01 ~]# ceph --admin-daemon /var/run/ceph/ceph-osd.0.asok help
```

### 删除存储池

```
[cephadm@ceph-admin ~]$ ceph tell mon.* injectargs --mon-allow-pool-delete=true
[cephadm@ceph-admin ~]$ ceph osd pool rm mypool mypool --yes-i-really-really-mean-it
[cephadm@ceph-admin ~]$ ceph tell mon.* injectargs --mon-allow-pool-delete=false
```

### 存储池配额

```
[cephadm@ceph-admin ~]$ ceph osd pool get-quota rbdpool
[cephadm@ceph-admin ~]$ ceph osd pool set-quota rbdpool max_objects 10000
set-quota max_objects = 10000 for pool rbdpool
[cephadm@ceph-admin ~]$ ceph osd pool get-quota rbdpool
quotas for pool 'rbdpool':
  max objects: 10 k objects
  max bytes  : N/A

```

### 添加用户

```
[cephadm@ceph-admin ~]$ ceph auth add client.testuser mon 'allow r' osd 'allow rw pool=rdbpool'
```

### 获取用户的信息

```
[cephadm@ceph-admin ~]$ ceph auth get client.testuser
```

### 列出用户的key

```
[cephadm@ceph-admin ~]$ ceph auth print-key client.testuser
```

### 更新权限

```
[cephadm@ceph-admin ~]$ ceph auth caps client.testuser mon 'allow rw' osd 'allow rw pool=rbdpool'
[cephadm@ceph-admin ~]$ ceph auth get client.testuser
```

### 删除用户

```
[cephadm@ceph-admin ~]$ ceph auth del client.testuser
```

### 创建keyring

```
[cephadm@ceph-admin ceph-cluster]$ ceph auth get-or-create client.kube mon 'allow r' osd 'allow * pool=kube'
[cephadm@ceph-admin ceph-cluster]$ ceph auth get client.kube
[cephadm@ceph-admin ceph-cluster]$ ceph auth get client.kube -o ceph.client.kube.keyring

```

### 合并keyring

```
[cephadm@ceph-admin ceph-cluster]$ ceph-authtool --create-keyring cluster.keyring
[cephadm@ceph-admin ceph-cluster]$ ceph-authtool cluster.keyring --import-keyring ./ceph.client.kube.keyring
[cephadm@ceph-admin ceph-cluster]$ ceph-authtool -l cluster.keyring
```

## 创建kube存储池

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool create kube 64 64
```

### 启用rbd功能

```
[cephadm@ceph-admin ceph-cluster]$ ceph osd pool application enable kube rbd
```

### 初始化rbd

```
[cephadm@ceph-admin ceph-cluster]$ rbd pool init kube
```

### 创建映像文件

```
[cephadm@ceph-admin ceph-cluster]$ rbd create --pool kube --image vol01 --size 2G
[cephadm@ceph-admin ceph-cluster]$ rbd ls --pool kube
[cephadm@ceph-admin ceph-cluster]$ rbd create --size 2G kube/vol02

```

```
[cephadm@ceph-admin ceph-cluster]$ rbd ls --pool kube -l --format json  --pretty-format
[
    {
        "image": "vol01",
        "size": 2147483648,
        "format": 2
    },
    {
        "image": "vol02",
        "size": 2147483648,
        "format": 2
    }
]

```

```
[cephadm@ceph-admin ceph-cluster]$ rbd info kube/vol01
[cephadm@ceph-admin ceph-cluster]$ rbd info --pool kube --image vol02
[cephadm@ceph-admin ceph-cluster]$ rbd info --pool kube vol01
```

### 禁用rbd的模型特性

```
[cephadm@ceph-admin ceph-cluster]$ rbd feature disable kube/vol01 object-map fast-diff deep-flatten
```

## 使用客户端连接

```
[root@mon02 ~]# scp /etc/yum.repos.d/ceph.repo 192.168.0.115:/etc/yum.repos.d/
[root@stor05 yum.repos.d]# yum -y install epel*
[root@stor05 yum.repos.d]# yum install ceph-common
[cephadm@ceph-admin ceph-cluster]$ scp ceph.conf root@192.168.190.115:/etc/ceph/
[root@stor05 ceph]# ceph --user kube -s
  cluster:
    id:     903285d9-d8c2-478b-85cb-70ca536eb33a
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum mon01,mon02,mon03
    mgr: mon03(active), standbys: stor04
    mds: cephfs-1/1/1 up  {0=mon02=up:active}
    osd: 8 osds: 8 up, 8 in
    rgw: 1 daemon active

  data:
    pools:   8 pools, 256 pgs
    objects: 221  objects, 3.8 KiB
    usage:   8.2 GiB used, 44 GiB / 52 GiB avail
    pgs:     256 active+clean

```

### 磁盘映射

```
[root@stor05 ceph]# rbd --user kube map kube/vol01
/dev/rbd0
[root@stor05 ceph]# fdisk -l
Disk /dev/rbd0: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
```

### 格式化

```
[root@stor05 ceph]# mkfs.xfs /dev/rbd0
[root@stor05 ceph]# mount /dev/rbd0  /mnt
[root@stor05 ceph]# cd /mnt/
[root@stor05 mnt]# ls
[root@stor05 mnt]# cp /etc/issue .
[root@stor05 mnt]# ls
issue
[root@stor05 mnt]# df -h
Filesystem                      Size  Used Avail Use% Mounted on
/dev/mapper/centos_stor05-root   17G  1.2G   16G   7% /
devtmpfs                        901M     0  901M   0% /dev
tmpfs                           912M     0  912M   0% /dev/shm
tmpfs                           912M  8.8M  903M   1% /run
tmpfs                           912M     0  912M   0% /sys/fs/cgroup
/dev/sda1                      1014M  143M  872M  15% /boot
tmpfs                           183M     0  183M   0% /run/user/0
/dev/rbd0                       2.0G   33M  2.0G   2% /mnt

```

### 查看内核是否支持ceph

```
[root@stor05 ~]# modinfo ceph

```

### 卸载磁盘映射

```
[root@stor05 ~]# rbd unmap /dev/rbd0
[root@stor05 ~]# rbd showmapped
```

### 设备mapp

```
[root@stor05 ~]# rbd showmapped
id pool image snap device
0  kube vol01 -    /dev/rbd0

```

### 查看rbd是否映射

```
[cephadm@ceph-admin ceph-cluster]$ rbd ls -p kube -l
NAME   SIZE PARENT FMT PROT LOCK
vol01 2 GiB          2      excl
vol02 2 GiB          2
```

### 空间扩展

```
[cephadm@ceph-admin ceph-cluster]$ rbd resize -s 5G kube/vol01
Resizing image: 100% complete...done.
[cephadm@ceph-admin ceph-cluster]$ rbd ls -p kube -l
NAME   SIZE PARENT FMT PROT LOCK
vol01 5 GiB          2
vol02 2 GiB          2
```

## 删除images

```
[cephadm@ceph-admin ceph-cluster]$ rbd rm kube/vol02
Removing image: 100% complete...done.
[cephadm@ceph-admin ceph-cluster]$ rbd ls -p kube -l
NAME   SIZE PARENT FMT PROT LOCK
vol01 5 GiB          2

```

### 移动到回收站

```
[cephadm@ceph-admin ceph-cluster]$ rbd trash move kube/vol01
[cephadm@ceph-admin ceph-cluster]$ rbd ls -p kube -l

```

### 查看回收站里面的内容

```
[cephadm@ceph-admin ceph-cluster]$ rbd trash list -p kube
20c9d6b8b4567 vol01

```

### 恢复回收站里面的image

```
[cephadm@ceph-admin ceph-cluster]$ rbd trash list -p kube
20c9d6b8b4567 vol01
[cephadm@ceph-admin ceph-cluster]$ rbd trash restore -p kube --image vol01 --image-id 20c9d6b8b4567
[cephadm@ceph-admin ceph-cluster]$ rbd ls -p kube
vol01
```

