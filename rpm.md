

# 软件管理基础

**软件运行环境**

**软件包基础**

**rpm包管理**

**yum管理**

**定制yum仓库**

**dnf管理**

**编译安装**

**Ubuntu软件管理**



**软件运行和编译**

ABI： Application Binary Interface

​       Windows与Linux不兼容

​              ELF（Executable and Linkeble Format)

​              PE (Protable Executable)

​         库级别的虚拟化：

​              Linux : WINE

​              Windows：Cygwin

API：Application Programming Interface

​		POSIX: Portable OS



程序源代码 --》预处理--》编译 --》 汇编 --》链接

​      	静态编译： .a

​		  动态编译：.so

查看系统中lib64下面的库文件

```
[root@shaoxh ~]# ls /lib64/
audit                               libdhcpctl.so.0               libgpgme-pthread.so.11.8.1  libmenuw.so.5                     libparted.so.2                libsystemd-daemon.so.0
cracklib_dict.hwm                   libdhcpctl.so.0.0.0           libgpgme.so.11              libmenuw.so.5.9                   libparted.so.2.0.0            libsystemd-daemon.so.0.0.12
cracklib_dict.pwd                   libdl-2.17.so                 libgpgme.so.11.8.1          libmnl.so.0                       libpciaccess.so.0             libsystemd-id128.so.0


```

ldd 命令查看cat命令需要的库文件

```
[root@shaoxh ~]# ldd /bin/cat
	linux-vdso.so.1 =>  (0x00007ffe753eb000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f8c0d0cf000)
	/lib64/ld-linux-x86-64.so.2 (0x00005559b8212000)

```

ldd 命令查看ls命令需要的库文件

```
[root@shaoxh ~]# ldd /bin/ls
	linux-vdso.so.1 =>  (0x00007fff3c1d1000)
	libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f4a5c626000)
	libcap.so.2 => /lib64/libcap.so.2 (0x00007f4a5c421000)
	libacl.so.1 => /lib64/libacl.so.1 (0x00007f4a5c217000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f4a5be54000)
	libpcre.so.1 => /lib64/libpcre.so.1 (0x00007f4a5bbf2000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007f4a5b9ed000)
	/lib64/ld-linux-x86-64.so.2 (0x0000561be4f12000)
	libattr.so.1 => /lib64/libattr.so.1 (0x00007f4a5b7e8000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f4a5b5cc000)

```

ldd 命令查看cp命令需要的库文件

```
[root@shaoxh ~]# ldd /bin/cp
	linux-vdso.so.1 =>  (0x00007fffe4f2c000)
	libselinux.so.1 => /lib64/libselinux.so.1 (0x00007fd291327000)
	libacl.so.1 => /lib64/libacl.so.1 (0x00007fd29111e000)
	libattr.so.1 => /lib64/libattr.so.1 (0x00007fd290f18000)
	libc.so.6 => /lib64/libc.so.6 (0x00007fd290b55000)
	libpcre.so.1 => /lib64/libpcre.so.1 (0x00007fd2908f3000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007fd2906ee000)
	/lib64/ld-linux-x86-64.so.2 (0x0000555a7566f000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fd2904d2000)

```

**静态和动态链接**

链接主要作用是把各模块之间相互引用的部分处理好，使得各个模块之间能够正确的衔接，分为动态链接和静态链接

**静态链接**	

​		把程序对应依赖库复制一份到包

​		libxxx.a

​		嵌入程序包

​		升级难，需重新编译

​		占用较多空间，迁移容易

**动态链接**

​		只把依赖加做一个动态链接

​		libxxx.so

​		连接指向

​		占用较少空间，升级方便

**开发语言**

​	系统级开发语言

​        C

​         C++

​	应用级开发

​	     java

​		Python

​		go

​		php

​		perl

​	    delphi

​	    ruby

**包和包管理器**

​		最初只有.tar.gz的打包的源码文件，用户必须编译每个他想在GUN/Linux上运行的软件。用户们急需系统提供一种方法来管理这些安装在机器上的软件，当Debian诞生时，这样一个管理工具也就应运而生，它被命名为dpkg。从而著名的“package”概念第一次出现在GUN/Linux系统中，稍后Red Hat才开发自己的“rpm”包管理系统。

**包的组成**

​		二进制文件、库文件、配置文件、帮助文件

**程序包管理器**

​	   debian : deb文件，dpkg包管理器

​	   redhat： rpm文件,rpm包管理器

​	                     rpm： Redhat Package Manager

**包命名**

​      源代码：name-VERSION.tar.gz|bz2|xz 

​                       VERSION: major.minor.release 

​      rpm包命名方式： 

​                       name-VERSION-release.arch.rpm 

​                例：bash-4.2.46-19.el7.x86_64.rpm 

​                        VERSION: major.minor.release 

​						 release：release.OS 

​    常见的arch： 

​                         x86: i386, i486, i586, i686 

​                         x86_64: x64, x86_64, amd64 

​                         powerpc: ppc 

​                         跟平台无关：noarch 



安装autofs包

```
[root@bogon ~]# systemctl start autofs
[root@bogon ~]# ls /misc/
[root@bogon ~]# 
[root@bogon ~]# 
[root@bogon ~]# cd /misc/
[root@bogon misc]# ls
[root@bogon misc]# 
[root@bogon misc]# 
[root@bogon misc]# cd cd
[root@bogon cd]# ls
CentOS_BuildTag  EFI  EULA  GPL  images  isolinux  LiveOS  Packages  repodata  RPM-GPG-KEY-CentOS-7  RPM-GPG-KEY-CentOS-Testing-7  TRANS.TBL
[root@bogon cd]# df 
Filesystem              1K-blocks    Used Available Use% Mounted on
/dev/mapper/centos-root  17811456 3274588  14536868  19% /
devtmpfs                   991328       0    991328   0% /dev
tmpfs                     1007240       0   1007240   0% /dev/shm
tmpfs                     1007240    9832    997408   1% /run
tmpfs                     1007240       0   1007240   0% /sys/fs/cgroup
/dev/sda1                 1038336  182332    856004  18% /boot
tmpfs                      201448      12    201436   1% /run/user/992
tmpfs                      201448       0    201448   0% /run/user/0
/dev/sr0                  8490330 8490330         0 100% /misc/cd
[root@bogon cd]# 
*
```

列出Package包里各平台包的数量

```
[root@bogon Packages]# ls *.rpm | rev|cut -d'.' -f2|sort |uniq -c|rev
x86_64 4734   
i686 1412   
noarch 6703   

```

```
[root@bogon Packages]# ls *.rpm | sed -r 's/.*\.(.*).rpm$/\1/' |sort |uniq -c
   2141 i686
   3076 noarch
   4374 x86_64

```

**包命名和工具**

​		包：分类和拆包

​                Application-VERSION-ARCH.rpm: 主包 

​               Application-devel-VERSION-ARCH.rpm 开发子包 

​               Application-utils-VERSION-ARHC.rpm 其它子包 

​               Application-libs-VERSION-ARHC.rpm 其它子包 

包之间：可能存在依赖关系，甚至循环依赖 

解决依赖包管理工具： 

​                yum：rpm包管理器的前端工具  

​                apt：deb包管理器前端工具  

​                zypper：suse上的rpm前端管理工具  

​                dnf：Fedora 18+ rpm包管理器前端管理工具 

安装httpd包

```
[root@bogon Packages]# rpm -ivh httpd-2.4.6-67.el7.centos.x86_64.rpm 
error: Failed dependencies:
	/etc/mime.types is needed by httpd-2.4.6-67.el7.centos.x86_64
	httpd-tools = 2.4.6-67.el7.centos is needed by httpd-2.4.6-67.el7.centos.x86_64
	libapr-1.so.0()(64bit) is needed by httpd-2.4.6-67.el7.centos.x86_64
	libaprutil-1.so.0()(64bit) is needed by httpd-2.4.6-67.el7.centos.x86_64

```

测试移动共享库

```
[root@bogon Packages]# ldd /bin/cat
	linux-vdso.so.1 =>  (0x00007ffd5f3b6000)
	libc.so.6 => /lib64/libc.so.6 (0x00007fcaf0bad000)
	/lib64/ld-linux-x86-64.so.2 (0x000055c082ae9000)
[root@bogon Packages]# mv /lib64/libc.so.6 /root/
[root@bogon Packages]# cat /etc/issue
cat: error while loading shared libraries: libc.so.6: cannot open shared object file: No such file or directory
[root@bogon Packages]# 
[root@bogon Packages]# cat /etc/issue /root
cat: error while loading shared libraries: libc.so.6: cannot open shared object file: No such file or directory
[root@bogon Packages]# mv /root/libc.so.6 /lib64/
mv: error while loading shared libraries: libc.so.6: cannot open shared object file: No such file or directory


```

光盘加载救援模式

```
1）Continue
原来的系统在 /mnt/sysimage
mv /mnt/sysimage/root/libc.so.6 /mnt/sysimage/lib64
```

**库文件**

查看二级制程序所依赖的库文件

​	    ldd /PATH/TO/BINARY_FILE

管理及查看本机装载的库文件

​	   ldconfig        加载配置文件中指定的库文件

/sbin/ldconfig -p  显示本机已经缓存的所有可用库文件名及文件路径映射关系

配置文件：/etc/ld.so.conf, /etc/ld.so.conf.d/*.conf

缓存文件: /etc/ld.so.cache

已经加载的使用的共享库

```
[root@bogon ~]# /sbin/ldconfig -p |wc -l
906

```

**包管理器**

 程序包管理器：

​      功能：将编译好的应用程序的各组成文件打包一个或几个程序包文件，从而方便快捷地实现程序包的安装、卸载、查               询、升级和校验等管理操作

包文件组成（每个包独有）

​	  RPM包内的文件

​      RPM 的元数据，如名称，版本、依赖性，描述等

​       安装或卸载时运行的脚本

数据库（公共）：/var/lib/rpm

​         程序包名称及版本

​         依赖关系

​          功能说明

​          包安装后生成的各文件路径及校验码信息



路径/var/lib/rpm

```
[root@bogon rpm]# pwd
/var/lib/rpm
[root@bogon rpm]# file *
Basenames:    Berkeley DB (Btree, version 9, native byte-order)
Conflictname: Berkeley DB (Btree, version 9, native byte-order)
__db.001:     Applesoft BASIC program data
__db.002:     386 pure executable not stripped
__db.003:     386 pure executable
Dirnames:     Berkeley DB (Btree, version 9, native byte-order)
Group:        Berkeley DB (Btree, version 9, native byte-order)
Installtid:   Berkeley DB (Btree, version 9, native byte-order)
Name:         Berkeley DB (Btree, version 9, native byte-order)
Obsoletename: Berkeley DB (Btree, version 9, native byte-order)
Packages:     Berkeley DB (Hash, version 9, native byte-order)
Providename:  Berkeley DB (Btree, version 9, native byte-order)
Requirename:  Berkeley DB (Btree, version 9, native byte-order)
Sha1header:   Berkeley DB (Btree, version 9, native byte-order)
Sigmd5:       Berkeley DB (Btree, version 9, native byte-order)
Triggername:  Berkeley DB (Btree, version 9, native byte-order)

```

**程序包的来源**

管理程序包的方式

​		使用包管理器：rpm

​	    使用前端工具：yum，dnf

获取程序包的途径：

​	1.系统发行版的光盘或官方的服务器

​	   CentOS镜像

​	    https://www.centos.org/download

​         http://mirrors.aliyun.com

​        http://mirrors.sohu.com

​         http://mirrors.163.com

2.项目官方站点



3.第三方组织

​		Fedora-EPEL

​              Extra  Packages  for  Enterprise Linux

​         Rpmforge:RHEL推荐，包很全 

搜索引擎： 

​          http://pkgs.org

​          http://rpmfind.net

​          http://rpmpbone.net

​         https://sourceforge.net

4.自己制作

注意：第三方包建议要要检查其合法性

​      来源合法性，程序包的完整性



**rpm包管理**

CentOS系统上使用rpm命令管理程序包

​    安装、卸载、升级、查询、校验、数据库维护

​	安装：

​	rpm {-i| install } [install-options]  PACKAGE _FILE ...

​             -v: verbose

​              -vv:

​             -h: 以#显示程序包管理执行进度

​    rpm -ivh PACKAGE _file 

安装mariadb数据

```
[root@bogon Packages]# rpm -ivh mariadb-5.5.56-2.el7.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:mariadb-1:5.5.56-2.el7           ################################# [100%]

```

```
[root@bogon Packages]# rpm -ivh mariadb-server-5.5.56-2.el7.x86_64.rpm 
error: Failed dependencies:
	perl-DBD-MySQL is needed by mariadb-server-1:5.5.56-2.el7.x86_64
[root@bogon Packages]# rpm -ivh perl
Display all 376 possibilities? (y or n)
[root@bogon Packages]# rpm -ivh perl-DBD-MySQL-4.023-5.el7.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:perl-DBD-MySQL-4.023-5.el7       ################################# [100%]
[root@bogon Packages]# rpm -ivh mariadb-server-5.5.56-2.el7.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:mariadb-server-1:5.5.56-2.el7    ################################# [100%]
[root@bogon Packages]# systemctl start mariadb

```

每次安装的rpm都会写入rpm目录下的数据库，时间发生改变

```
[root@bogon Packages]# ll /var/lib/rpm
total 81044
-rw-r--r--. 1 root root  3571712 Aug 27 00:37 Basenames
-rw-r--r--. 1 root root    16384 Aug 26 22:01 Conflictname
-rw-r--r--. 1 root root   270336 Aug 27 00:39 __db.001
-rw-r--r--. 1 root root    81920 Aug 27 00:39 __db.002
-rw-r--r--. 1 root root  1318912 Aug 27 00:39 __db.003
-rw-r--r--. 1 root root  1069056 Aug 27 00:37 Dirnames
-rw-r--r--. 1 root root    32768 Aug 27 00:37 Group
-rw-r--r--. 1 root root    20480 Aug 27 00:37 Installtid
-rw-r--r--. 1 root root    69632 Aug 27 00:37 Name
-rw-r--r--. 1 root root    32768 Aug 27 00:37 Obsoletename
-rw-r--r--. 1 root root 73592832 Aug 27 00:37 Packages
-rw-r--r--. 1 root root  2355200 Aug 27 00:37 Providename
-rw-r--r--. 1 root root   483328 Aug 27 00:37 Requirename
-rw-r--r--. 1 root root   126976 Aug 27 00:37 Sha1header
-rw-r--r--. 1 root root    73728 Aug 27 00:37 Sigmd5
-rw-r--r--. 1 root root     8192 Aug 26 22:01 Triggername

```

[install-options] 安装选项：

--test: 测试安装，但不真正执行安装，即dry run模式 

```
[root@bogon Packages]# rpm -ivh --test lftp-4.4.8-8.el7_3.2.x86_64.rpm 
Preparing...                          ################################# [100%]
[root@bogon Packages]# rpm -q lftp
package lftp is not installed
[root@bogon Packages]# 
[root@bogon Packages]# rpm -ivh lftp-4.4.8-8.el7_3.2.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:lftp-4.4.8-8.el7_3.2             ################################# [100%]
[root@bogon Packages]# rpm -q lftp
lftp-4.4.8-8.el7_3.2.x86_64

```

--nodeps:忽略依赖关系 

```
[root@bogon Packages]# rpm -ivh httpd-2.4.6-67.el7.centos.x86_64.rpm 
error: Failed dependencies:
	/etc/mime.types is needed by httpd-2.4.6-67.el7.centos.x86_64
	httpd-tools = 2.4.6-67.el7.centos is needed by httpd-2.4.6-67.el7.centos.x86_64
	libapr-1.so.0()(64bit) is needed by httpd-2.4.6-67.el7.centos.x86_64
	libaprutil-1.so.0()(64bit) is needed by httpd-2.4.6-67.el7.centos.x86_64
[root@bogon Packages]# rpm -ivh httpd-2.4.6-67.el7.centos.x86_64.rpm ^C
[root@bogon Packages]# 
[root@bogon Packages]# 
[root@bogon Packages]# rpm -ivh --nodeps httpd-2.4.6-67.el7.centos.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:httpd-2.4.6-67.el7.centos        ################################# [100%]
[root@bogon Packages]# 

```

-ql：查看包的文件列表

```
[root@bogon Packages]# which tree
/usr/bin/tree
[root@bogon Packages]# 
[root@bogon Packages]# 
[root@bogon Packages]# rpm -ql tree
/usr/bin/tree
/usr/share/doc/tree-1.6.0
/usr/share/doc/tree-1.6.0/LICENSE
/usr/share/doc/tree-1.6.0/README
/usr/share/man/man1/tree.1.gz

```

--replacepkgs:覆盖安装

```
[root@bogon Packages]# rm -f /usr/bin/tree 
[root@bogon Packages]# rpm -i tree-1.6.0-10.el7.x86_64.rpm 
	package tree-1.6.0-10.el7.x86_64 is already installed
[root@bogon Packages]# 
[root@bogon Packages]# rpm -ivh --replacepkgs tree-1.6.0-10.el7.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:tree-1.6.0-10.el7                ################################# [100%]

```

--replacefiles:覆盖冲突的文件安装，两个包同一个软件，版本不一样

-qi ：查询包的信息 （query information）

```
[root@bogon Packages]# rpm -qi bash
Name        : bash
Version     : 4.2.46
Release     : 28.el7
Architecture: x86_64
Install Date: Wed 26 Aug 2020 09:45:30 PM CST
Group       : System Environment/Shells
Size        : 3663637
License     : GPLv3+
Signature   : RSA/SHA256, Thu 10 Aug 2017 11:03:40 PM CST, Key ID 24c6a8a7f4a80eb5
Source RPM  : bash-4.2.46-28.el7.src.rpm
Build Date  : Thu 03 Aug 2017 05:13:21 AM CST
Build Host  : c1bm.rdu2.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://www.gnu.org/software/bash
Summary     : The GNU Bourne Again shell
Description :
The GNU Bourne Again shell (Bash) is a shell or command language
interpreter that is compatible with the Bourne shell (sh). Bash
incorporates useful features from the Korn shell (ksh) and the C shell
(csh). Most sh scripts can be run by bash without modification.

```

rpm包也可以使用cpio解开：

```
[root@bogon Packages]# cp tree-1.6.0-10.el7.x86_64.rpm /root/
[root@bogon Packages]# cd /root
[root@bogon ~]# ls
anaconda-ks.cfg  initial-setup-ks.cfg  tree-1.6.0-10.el7.x86_64.rpm
[root@bogon ~]# rpm2cpio tree-1.6.0-10.el7.x86_64.rpm |cpio -tv
-rwxr-xr-x   1 root     root        62768 Jun 10  2014 ./usr/bin/tree
drwxr-xr-x   2 root     root            0 Jun 10  2014 ./usr/share/doc/tree-1.6.0
-rw-r--r--   1 root     root        18009 Aug 13  2004 ./usr/share/doc/tree-1.6.0/LICENSE
-rw-r--r--   1 root     root         4628 Jun 24  2011 ./usr/share/doc/tree-1.6.0/README
-rw-r--r--   1 root     root         4100 Jun 24  2011 ./usr/share/man/man1/tree.1.gz
177 blocks
[root@bogon ~]# rpm2cpio tree-1.6.0-10.el7.x86_64.rpm |cpio -idv ./usr/bin/tree
./usr/bin/tree
177 blocks
[root@bogon ~]# tree usr/
usr/
└── bin
    └── tree

1 directory, 1 file
[root@bogon ~]# mv usr/bin/tree /bin 

```

--nosignature: 不检查来源合法性 

```
[root@bogon ~]# rpm -ivh tree-1.6.0-10.el7.x86_64.rpm --nosignature
Preparing...                          ################################# [100%]
	package tree-1.6.0-10.el7.x86_64 is already installed

```

--nodigest：不检查包完整性 



--noscripts：不执行程序包脚本 

​             %pre: 安装前脚本   --nopre  

​             %post: 安装后脚本   --nopost  

​              %preun: 卸载前脚本 --nopreun   

​              %postun: 卸载后脚本  --nopostun 

```
[root@bogon ~]# rpm -q --scripts bash
postinstall scriptlet (using <lua>):
nl        = '\n'
sh        = '/bin/sh'..nl
bash      = '/bin/bash'..nl
f = io.open('/etc/shells', 'a+')
if f then
  local shells = nl..f:read('*all')..nl
  if not shells:find(nl..sh) then f:write(sh) end
  if not shells:find(nl..bash) then f:write(bash) end
  f:close()
end
postuninstall scriptlet (using <lua>):
-- Run it only if we are uninstalling
if arg[2] == "0"
then
  t={}
  for line in io.lines("/etc/shells")
  do
    if line ~= "/bin/bash" and line ~= "/bin/sh"
    then
      table.insert(t,line)
    end
  end

  f = io.open("/etc/shells", "w+")
  for n,line in pairs(t)
  do
    f:write(line.."\n")
  end
  f:close()
end
[root@bogon ~]# 

```

如果没有安装的rpm，需要加上-p查询

```
[root@bogon Packages]# rpm -q httpd
package httpd is not installed
[root@bogon Packages]# rpm -qp --scripts httpd-2.4.6-67.el7.centos.x86_64.rpm 
preinstall scriptlet (using /bin/sh):
# Add the "apache" group and user
/usr/sbin/groupadd -g 48 -r apache 2> /dev/null || :
/usr/sbin/useradd -c "Apache" -u 48 -g apache \
	-s /sbin/nologin -r -d /usr/share/httpd apache 2> /dev/null || :
postinstall scriptlet (using /bin/sh):

if [ $1 -eq 1 ] ; then 
        # Initial installation 
        systemctl preset httpd.service htcacheclean.service >/dev/null 2>&1 || : 
fi
preuninstall scriptlet (using /bin/sh):

if [ $1 -eq 0 ] ; then 
        # Package removal, not upgrade 
        systemctl --no-reload disable httpd.service htcacheclean.service > /dev/null 2>&1 || : 
        systemctl stop httpd.service htcacheclean.service > /dev/null 2>&1 || : 
fi
postuninstall scriptlet (using /bin/sh):

systemctl daemon-reload >/dev/null 2>&1 || : 


# Trigger for conversion from SysV, per guidelines at:
# https://fedoraproject.org/wiki/Packaging:ScriptletSnippets#Systemd
posttrans scriptlet (using /bin/sh):
test -f /etc/sysconfig/httpd-disable-posttrans || \
  /bin/systemctl try-restart httpd.service htcacheclean.service >/dev/null 2>&1 || :

```

**rpm包升级**

升级： 

rpm {-U|--upgrade} [install-options] PACKAGE_FILE... 

rpm {-F|--freshen} [install-options] PACKAGE_FILE... 

upgrade：安装有旧版程序包，则“升级” 

​                   如果不存在旧版程序包，则“安装” 

freshen：安装有旧版程序包，则“升级” 

​                   如果不存在旧版程序包，则不执行升级操作 

rpm -Uvh PACKAGE_FILE ... 

rpm -Fvh PACKAGE_FILE ... 

--oldpackage：降级 

--force: 强制安装 



**包查询**

rpm {-q|--query} [select-options] [query-options] 

[select-options] 

​     -a：所有包 

```
[root@bogon Packages]# rpm -qa |wc -l
1263
[root@bogon Packages]# rpm -qa |grep maria
mariadb-libs-5.5.56-2.el7.x86_64
mariadb-5.5.56-2.el7.x86_64
mariadb-server-5.5.56-2.el7.x86_64
[root@bogon Packages]# rpm -qa "mari*"
marisa-0.2.4-4.el7.x86_64
mariadb-libs-5.5.56-2.el7.x86_64
mariadb-5.5.56-2.el7.x86_64
mariadb-server-5.5.56-2.el7.x86_64

```

  	-f：查看指定的文件由哪个程序包安装生成 

```
root@bogon ~]# rpm -ql tree
/usr/bin/tree
/usr/share/doc/tree-1.6.0
/usr/share/doc/tree-1.6.0/LICENSE
/usr/share/doc/tree-1.6.0/README
/usr/share/man/man1/tree.1.gz
[root@bogon ~]# rpm -qf /usr/bin/tree
tree-1.6.0-10.el7.x86_64

```

​	-p rpmfile：针对尚未安装的程序包文件做查询操作 

```
[root@bogon Packages]# rpm -qpi tree-1.6.0-10.el7.x86_64.rpm 
Name        : tree
Version     : 1.6.0
Release     : 10.el7
Architecture: x86_64
Install Date: (not installed)
Group       : Applications/File
Size        : 89505
License     : GPLv2+
Signature   : RSA/SHA256, Fri 04 Jul 2014 01:36:46 PM CST, Key ID 24c6a8a7f4a80eb5
Source RPM  : tree-1.6.0-10.el7.src.rpm
Build Date  : Tue 10 Jun 2014 03:28:53 AM CST
Build Host  : worker1.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://mama.indstate.edu/users/ice/tree/
Summary     : File system tree viewer
Description :
The tree utility recursively displays the contents of directories in a
tree-like format.  Tree is basically a UNIX port of the DOS tree
utility.

```

​	--whatprovides CAPABILITY：查询指定的CAPABILITY由哪个包所提供 

```
[root@bogon Packages]# rpm -q --whatprovides /bin/bash
bash-4.2.46-28.el7.x86_64
[root@bogon Packages]# rpm -q --whatprovides /usr/bin/tree
tree-1.6.0-10.el7.x86_64

```

​	--whatrequires CAPABILITY：查询指定的CAPABILITY被哪个包所依赖 

```
[root@shaoxh ~]# rpm -q --whatrequires /bin/bash
nss-softokn-freebl-3.28.3-6.el7.x86_64
iproute-3.10.0-87.el7.x86_64
grubby-8.28-23.el7.x86_64
openssl-1.0.2k-8.el7.x86_64
rpm-4.11.3-25.el7.x86_64
openldap-2.4.44-5.el7.x86_64
policycoreutils-2.5-17.1.el7.x86_64
teamd-1.25-5.el7.x86_64
device-mapper-1.02.140-8.el7.x86_64
dracut-033-502.el7.x86_64
kmod-20-15.el7.x86_64
systemd-219-42.el7.x86_64
initscripts-9.49.39-1.el7.x86_64
crontabs-1.11-6.20121102git.el7.noarch
plymouth-scripts-0.8.9-0.28.20140113.el7.centos.x86_64
dhclient-4.2.5-58.el7.centos.x86_64
dracut-network-033-502.el7.x86_64
ebtables-2.0.10-15.el7.x86_64
firewalld-0.4.4.4-6.el7.noarch
kexec-tools-2.0.14-17.el7.x86_64
lvm2-2.02.171-8.el7.x86_64
audit-2.7.6-3.el7.x86_64
postfix-2.10.1-6.el7.x86_64
openssh-server-7.4p1-11.el7.x86_64
chrony-3.1-2.el7.centos.x86_64
dracut-config-rescue-033-502.el7.x86_64
iprutils-2.4.14.1-1.el7.x86_64
man-db-2.6.3-9.el7.x86_64
selinux-policy-targeted-3.13.1-166.el7.noarch

```

rpm2cpio 包文件|cpio –itv  预览包内文件 

rpm2cpio 包文件|cpio –id  “*.conf” 释放包内文件 

--changelog：查询rpm包的changelog 

```
[root@shaoxh ~]# rpm -q --changelog bash

```

-c：查询程序的配置文件 

```
[root@shaoxh ~]# rpm -qc bash
/etc/skel/.bash_logout
/etc/skel/.bash_profile
/etc/skel/.bashrc

```

-d：查询程序的文档 

```
[root@shaoxh ~]# rpm -qd bash
/usr/share/doc/bash-4.2.46/COPYING
/usr/share/info/bash.info.gz
/usr/share/man/man1/..1.gz
/usr/share/man/man1/:.1.gz
/usr/share/man/man1/[.1.gz
/usr/share/man/man1/alias.1.gz
/usr/share/man/man1/bash.1.gz
/usr/share/man/man1/bashbug-64.1.gz
/usr/share/man/man1/bashbug.1.gz

```

-i：information 

-l：查看指定的程序包安装后生成的所有文件 

--scripts：程序包自带的脚本 

--provides：列出指定程序包所提供的CAPABILITY 

```
[root@shaoxh ~]# rpm -q --provides bash
/bin/bash
/bin/sh
bash = 4.2.46-28.el7
bash(x86-64) = 4.2.46-28.el7
config(bash) = 4.2.46-28.el7

```

-R：查询指定的程序包所依赖的CAPABILITY 

```
[root@bogon Packages]# rpm -qpR httpd-2.4.6-67.el7.centos.x86_64.rpm 
/etc/mime.types
system-logos >= 7.92.1-1
httpd-tools = 2.4.6-67.el7.centos
/usr/sbin/useradd
/usr/sbin/groupadd
systemd-units
systemd-units
systemd-units
/bin/sh
/bin/sh
/bin/sh
/bin/sh
/bin/sh
/bin/sh
rpmlib(FileDigests) <= 4.6.0-1
rpmlib(FileCaps) <= 4.6.1-1
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
rpmlib(CompressedFileNames) <= 3.0.4-1
/bin/sh

```

常用查询用法：

-qi

-qf

-qc

-ql

-qd

-qpi

-qpl

-qa



包卸载： 

rpm {-e|--erase} [--allmatches] [--nodeps] [--noscripts] [--notriggers] [--test] PACKAGE_NAME ... 

当包卸载时，对应的配置文件不会删除， 以FILENAME.rpmsave形式保留 



rpm属于哪个包并且强行卸载

```
[root@shaoxh ~]# rpm -qf `which rpm`
rpm-4.11.3-25.el7.x86_64
[root@shaoxh ~]# 
[root@shaoxh ~]# 
[root@shaoxh ~]# rpm -e rpm --nodeps
[root@shaoxh ~]# 

```

如果想恢复，进入救援模式

```
rpm -ivh /run/install/repo/Packages/rpm-4.11.3-25.el7.x86_64.rpm --root=/mnt/sysimage
```

安装两个内核

```
rpm -ivh kernel-3.10.0-229.el7.x86_64.rpm --force
rpm -q kernel
rpm -e kernel-3.10.0-693.el7.x86_64 --nodeps

```

**包校验**

rpm {-V|--verify} [select-options] [verify-options] 

S file Size differs  

M Mode differs (includes permissions and file type) 

5 digest (formerly MD5 sum) differs 

 D Device major/minor number mismatch  

L readLink(2) path mismatch 

U User ownership differs  

G Group ownership differs  

T mTime differs 

 P capabilities differ 

包属性的改变可以查到

```
[root@bogon ~]# ll /bin/tree
-rwxr-xr-x. 1 root root 62768 Jun 10  2014 /bin/tree
[root@bogon ~]# useradd wang
[root@bogon ~]# chown wang /bin/tree
[root@bogon ~]# rpm -V tree
.....U...    /usr/bin/tree

```

恢复

```
[root@bogon ~]# chown root /bin/tree
[root@bogon ~]# rpm -V tree
[root@bogon ~]# 

```

大小的改变：

```
[root@bogon ~]# echo >> /bin/tree
[root@bogon ~]# rpm -V tree
S.5....T.    /usr/bin/tree
[root@bogon ~]# 

```

恢复：打开二进制文件

```
vim -b /bin/tree
:%!xxd
删除最后的oa
:%!xxd -r
:wd
[root@bogon ~]# rpm -V tree
.......T.    /usr/bin/tree
[root@bogon ~]# 

```

导入公钥：

```
[root@shaoxh cd]# rpm --import RPM-GPG-KEY-CentOS-7 
[root@shaoxh cd]# cat RPM-GPG-KEY-CentOS-7
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1.4.5 (GNU/Linux)

mQINBFOn/0sBEADLDyZ+DQHkcTHDQSE0a0B2iYAEXwpPvs67cJ4tmhe/iMOyVMh9
Yw/vBIF8scm6T/vPN5fopsKiW9UsAhGKg0epC6y5ed+NAUHTEa6pSOdo7CyFDwtn
4HF61Esyb4gzPT6QiSr0zvdTtgYBRZjAEPFVu3Dio0oZ5UQZ7fzdZfeixMQ8VMTQ
4y4x5vik9B+cqmGiq9AW71ixlDYVWasgR093fXiD9NLT4DTtK+KLGYNjJ8eMRqfZ
Ws7g7C+9aEGHfsGZ/SxLOumx/GfiTloal0dnq8TC7XQ/JuNdB9qjoXzRF+faDUsj
WuvNSQEqUXW1dzJjBvroEvgTdfCJfRpIgOrc256qvDMp1SxchMFltPlo5mbSMKu1
x1p4UkAzx543meMlRXOgx2/hnBm6H6L0FsSyDS6P224yF+30eeODD4Ju4BCyQ0jO
IpUxmUnApo/m0eRelI6TRl7jK6aGqSYUNhFBuFxSPKgKYBpFhVzRM63Jsvib82rY
438q3sIOUdxZY6pvMOWRkdUVoz7WBExTdx5NtGX4kdW5QtcQHM+2kht6sBnJsvcB
JYcYIwAUeA5vdRfwLKuZn6SgAUKdgeOtuf+cPR3/E68LZr784SlokiHLtQkfk98j
NXm6fJjXwJvwiM2IiFyg8aUwEEDX5U+QOCA0wYrgUQ/h8iathvBJKSc9jQARAQAB
tEJDZW50T1MtNyBLZXkgKENlbnRPUyA3IE9mZmljaWFsIFNpZ25pbmcgS2V5KSA8
c2VjdXJpdHlAY2VudG9zLm9yZz6JAjUEEwECAB8FAlOn/0sCGwMGCwkIBwMCBBUC
CAMDFgIBAh4BAheAAAoJECTGqKf0qA61TN0P/2730Th8cM+d1pEON7n0F1YiyxqG
QzwpC2Fhr2UIsXpi/lWTXIG6AlRvrajjFhw9HktYjlF4oMG032SnI0XPdmrN29lL
F+ee1ANdyvtkw4mMu2yQweVxU7Ku4oATPBvWRv+6pCQPTOMe5xPG0ZPjPGNiJ0xw
4Ns+f5Q6Gqm927oHXpylUQEmuHKsCp3dK/kZaxJOXsmq6syY1gbrLj2Anq0iWWP4
Tq8WMktUrTcc+zQ2pFR7ovEihK0Rvhmk6/N4+4JwAGijfhejxwNX8T6PCuYs5Jiv
hQvsI9FdIIlTP4XhFZ4N9ndnEwA4AH7tNBsmB3HEbLqUSmu2Rr8hGiT2Plc4Y9AO
aliW1kOMsZFYrX39krfRk2n2NXvieQJ/lw318gSGR67uckkz2ZekbCEpj/0mnHWD
3R6V7m95R6UYqjcw++Q5CtZ2tzmxomZTf42IGIKBbSVmIS75WY+cBULUx3PcZYHD
ZqAbB0Dl4MbdEH61kOI8EbN/TLl1i077r+9LXR1mOnlC3GLD03+XfY8eEBQf7137
YSMiW5r/5xwQk7xEcKlbZdmUJp3ZDTQBXT06vavvp3jlkqqH9QOE8ViZZ6aKQLqv
pL+4bs52jzuGwTMT7gOR5MzD+vT0fVS7Xm8MjOxvZgbHsAgzyFGlI1ggUQmU7lu3
uPNL0eRx4S1G4Jn5
=OGYX
-----END PGP PUBLIC KEY BLOCK-----

```

```
[root@shaoxh cd]# rpm -ivh Packages/tree-1.6.0-10.el7.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:tree-1.6.0-10.el7                ################################# [100%]

```



密码的存放位置：

```
[root@shaoxh cd]# rpm -qa "gpg-pubkey"
gpg-pubkey-f4a80eb5-53a7ff4b
[root@shaoxh cd]# 
[root@shaoxh cd]# 
[root@shaoxh cd]# rpm -qi "gpg-pubkey"
Name        : gpg-pubkey
Version     : f4a80eb5
Release     : 53a7ff4b
Architecture: (none)
Install Date: Thu 27 Aug 2020 12:16:44 AM EDT
Group       : Public Keys
Size        : 0
License     : pubkey
Signature   : (none)
Source RPM  : (none)
Build Date  : Mon 23 Jun 2014 06:19:55 AM EDT
Build Host  : localhost
Relocations : (not relocatable)
Packager    : CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>
Summary     : gpg(CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>)
Description :
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: rpm-4.11.3 (NSS-3)

mQINBFOn/0sBEADLDyZ+DQHkcTHDQSE0a0B2iYAEXwpPvs67cJ4tmhe/iMOyVMh9
Yw/vBIF8scm6T/vPN5fopsKiW9UsAhGKg0epC6y5ed+NAUHTEa6pSOdo7CyFDwtn
4HF61Esyb4gzPT6QiSr0zvdTtgYBRZjAEPFVu3Dio0oZ5UQZ7fzdZfeixMQ8VMTQ
4y4x5vik9B+cqmGiq9AW71ixlDYVWasgR093fXiD9NLT4DTtK+KLGYNjJ8eMRqfZ
Ws7g7C+9aEGHfsGZ/SxLOumx/GfiTloal0dnq8TC7XQ/JuNdB9qjoXzRF+faDUsj
WuvNSQEqUXW1dzJjBvroEvgTdfCJfRpIgOrc256qvDMp1SxchMFltPlo5mbSMKu1
x1p4UkAzx543meMlRXOgx2/hnBm6H6L0FsSyDS6P224yF+30eeODD4Ju4BCyQ0jO
IpUxmUnApo/m0eRelI6TRl7jK6aGqSYUNhFBuFxSPKgKYBpFhVzRM63Jsvib82rY
438q3sIOUdxZY6pvMOWRkdUVoz7WBExTdx5NtGX4kdW5QtcQHM+2kht6sBnJsvcB
JYcYIwAUeA5vdRfwLKuZn6SgAUKdgeOtuf+cPR3/E68LZr784SlokiHLtQkfk98j
NXm6fJjXwJvwiM2IiFyg8aUwEEDX5U+QOCA0wYrgUQ/h8iathvBJKSc9jQARAQAB
tEJDZW50T1MtNyBLZXkgKENlbnRPUyA3IE9mZmljaWFsIFNpZ25pbmcgS2V5KSA8
c2VjdXJpdHlAY2VudG9zLm9yZz6JAjUEEwECAB8FAlOn/0sCGwMGCwkIBwMCBBUC
CAMDFgIBAh4BAheAAAoJECTGqKf0qA61TN0P/2730Th8cM+d1pEON7n0F1YiyxqG
QzwpC2Fhr2UIsXpi/lWTXIG6AlRvrajjFhw9HktYjlF4oMG032SnI0XPdmrN29lL
F+ee1ANdyvtkw4mMu2yQweVxU7Ku4oATPBvWRv+6pCQPTOMe5xPG0ZPjPGNiJ0xw
4Ns+f5Q6Gqm927oHXpylUQEmuHKsCp3dK/kZaxJOXsmq6syY1gbrLj2Anq0iWWP4
Tq8WMktUrTcc+zQ2pFR7ovEihK0Rvhmk6/N4+4JwAGijfhejxwNX8T6PCuYs5Jiv
hQvsI9FdIIlTP4XhFZ4N9ndnEwA4AH7tNBsmB3HEbLqUSmu2Rr8hGiT2Plc4Y9AO
aliW1kOMsZFYrX39krfRk2n2NXvieQJ/lw318gSGR67uckkz2ZekbCEpj/0mnHWD
3R6V7m95R6UYqjcw++Q5CtZ2tzmxomZTf42IGIKBbSVmIS75WY+cBULUx3PcZYHD
ZqAbB0Dl4MbdEH61kOI8EbN/TLl1i077r+9LXR1mOnlC3GLD03+XfY8eEBQf7137
YSMiW5r/5xwQk7xEcKlbZdmUJp3ZDTQBXT06vavvp3jlkqqH9QOE8ViZZ6aKQLqv
pL+4bs52jzuGwTMT7gOR5MzD+vT0fVS7Xm8MjOxvZgbHsAgzyFGlI1ggUQmU7lu3
uPNL0eRx4S1G4Jn5
=OGYX
-----END PGP PUBLIC KEY BLOCK-----

```



卸载：安装程序会有警告

```
[root@shaoxh cd]# rpm -e  gpg-pubkey
[root@shaoxh cd]# rpm -qi "gpg-pubkey"
package gpg-pubkey is not installed

```

光盘和本地磁盘都有此公钥

```
[root@shaoxh cd]#   ls /etc/pki/rpm-gpg/
RPM-GPG-KEY-CentOS-7  RPM-GPG-KEY-CentOS-Debug-7  RPM-GPG-KEY-CentOS-Testing-7
[root@shaoxh cd]# diff /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 /misc/cd/RPM-GPG-KEY-CentOS-7 

```

导入后又具有验证功能

```
[root@shaoxh cd]# rpm -K Packages/tree-1.6.0-10.el7.x86_64.rpm 
Packages/tree-1.6.0-10.el7.x86_64.rpm: RSA sha1 ((MD5) PGP) md5 NOT OK (MISSING KEYS: (MD5) PGP#f4a80eb5) 
[root@shaoxh cd]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 
[root@shaoxh cd]# rpm -K Packages/tree-1.6.0-10.el7.x86_64.rpm 
Packages/tree-1.6.0-10.el7.x86_64.rpm: rsa sha1 (md5) pgp md5 OK

```

```
root@shaoxh ~]# cp /misc/cd/Packages/tree-1.6.0-10.el7.x86_64.rpm .
[root@shaoxh ~]# ls
anaconda-ks.cfg  tree-1.6.0-10.el7.x86_64.rpm  txt
[root@shaoxh ~]# 
[root@shaoxh ~]# echo >> tree-1.6.0-10.el7.x86_64.rpm 
[root@shaoxh ~]# rpm -K tree-1.6.0-10.el7.x86_64.rpm 
tree-1.6.0-10.el7.x86_64.rpm: rsa sha1 (MD5) PGP MD5 NOT OK

```

vim -b打开修改rpm包

```
[root@shaoxh ~]# vim -b tree-1.6.0-10.el7.x86_64.rpm 
[root@shaoxh ~]# 
[root@shaoxh ~]# 
[root@shaoxh ~]# 
[root@shaoxh ~]# 
[root@shaoxh ~]# 
[root@shaoxh ~]# rpm -K tree-1.6.0-10.el7.x86_64.rpm 
tree-1.6.0-10.el7.x86_64.rpm: rsa sha1 (md5) pgp md5 OK

```

**rpm数据库**

数据库重建

​    /var/lib/rpm

rpm {--initdb|--rebuilddb} 

initdb: 初始化   如果事先不存在数据库，则新建之   否则，不执行任何操作 

rebuilddb：重建已安装的包头的数据库索引目录 

```
[root@shaoxh ~]# mv /var/lib/rpm .
[root@shaoxh ~]# ls
rpm
[root@shaoxh ~]# 
[root@shaoxh ~]# rpm -qa
[root@shaoxh ~]# \mv rpm/* /var/lib/rpm
[root@shaoxh ~]# rpm -qa|grep tree
tree-1.6.0-10.el7.x86_64

```



**yum**

CentOS: yum, dnf

YUM: Yellowdog Update Modifier，rpm的前端程序，可解决软件包相关依 赖性，可在多个库之间定位软件包，up2date的替代工具

yum repository: yum repo，存储了众多rpm包，以及包的相关的元数据 文件（放置于特定目录repodata下） 

文件服务器：   

http://   

https://   

ftp://   

file://

**yum配置文件**

yum客户端配置文件：

​      /etc/yum.conf：为所有仓库提供公共配置 

​      /etc/yum.repos.d/*.repo：为仓库的指向提供配置 

仓库指向的定义： 

​        [repositoryID] 

​        name=Some name for this repository 

​         baseurl=url://path/to/repository/ 

​         enabled={1|0} 

​          gpgcheck={1|0} 

​          gpgkey=URL   

​          enablegroups={1|0} 

​           failovermethod={roundrobin|priority} 

​                     roundrobin：意为随机挑选，默认值 

​                    priority:按顺序访问 

​           cost=   默认为1000 

**yum仓库**

​           yum的repo配置文件中可用的变量：

​                     $releaserver:当前OS发行版的主版本号

​		           $arch:平台，i386,i486,i586,x86_64等

​	                $basearch:基础平台；i386,x86_64

​					$YUM-$YUM9:自定义变量

​            示例： 

​	                  http://server/centos/$releasever/$basearch/

​					http://server/centos/7/x86_64

​		            http://server/centos/6/i386            

配置客户端仓库

```
[root@shaoxh yum.repos.d]# cat centos.repo 
[base]
name=shaoxh
baseurl=file:///misc/cd
enabled=1
gpgcheck=0

```

配置epel仓库

```
[root@shaoxh yum.repos.d]# cat epel.repo 
[epel]
name=aliyun
baseurl=https://mirrors.aliyun.com/epel/7/x86_64/	
gpgcheck=0
enabled=1

```

清理仓库

```
[root@shaoxh yum.repos.d]# ll /var/cache/yum/x86_64/7/
total 8
drwxr-xr-x. 4 root root  256 Aug 27 02:18 base
drwxr-xr-x. 4 root root 4096 Aug 27 02:18 epel
drwxr-xr-x. 4 root root  183 Aug 27 00:16 extras
-rw-r--r--. 1 root root  166 Aug 27 02:18 timedhosts
-rw-r--r--. 1 root root    0 Aug 27 02:18 timedhosts.txt
drwxr-xr-x. 4 root root  183 Aug 27 00:16 updates
[root@shaoxh yum.repos.d]# 
[root@shaoxh yum.repos.d]# 
[root@shaoxh yum.repos.d]# 
[root@shaoxh yum.repos.d]# yum clean all
Loaded plugins: fastestmirror
Cleaning repos: base epel
Cleaning up everything
Maybe you want: rm -rf /var/cache/yum, to also free up space taken by orphaned data from disabled or removed repos
Cleaning up list of fastest mirrors
[root@shaoxh yum.repos.d]# ll /var/cache/yum/x86_64/7
total 4
drwxr-xr-x. 4 root root  33 Aug 27 02:21 base
drwxr-xr-x. 4 root root  33 Aug 27 02:21 epel
drwxr-xr-x. 4 root root 183 Aug 27 00:16 extras
-rw-r--r--. 1 root root 166 Aug 27 02:18 timedhosts
drwxr-xr-x. 4 root root 183 Aug 27 00:16 updates

```



```
[root@shaoxh yum.repos.d]# rm -rf /var/cache/yum/*
[root@shaoxh yum.repos.d]# yum repolist
Loaded plugins: fastestmirror
base                                                                                                                                                                                     
[root@shaoxh yum.repos.d]# ll /var/cache/yum/x86_64/7
total 8
drwxr-xr-x. 4 root root  256 Aug 27 02:23 base
drwxr-xr-x. 4 root root 4096 Aug 27 02:23 epel
-rw-r--r--. 1 root root   80 Aug 27 02:23 timedhosts
-rw-r--r--. 1 root root    0 Aug 27 02:23 timedhosts.txt

```

```
[root@shaoxh yum.repos.d]# yum makecache
Loaded plugins: fastestmirror
........
Metadata Cache Created
[root@shaoxh yum.repos.d]# ll /var/cache/yum/x86_64/7
total 12
drwxr-xr-x. 4 root root 4096 Aug 27 02:24 base
drwxr-xr-x. 4 root root 4096 Aug 27 02:24 epel
-rw-r--r--. 1 root root   79 Aug 27 02:24 timedhosts
-rw-r--r--. 1 root root    0 Aug 27 02:24 timedhosts.txt

```

老司机玩具：

```
[root@shaoxh yum.repos.d]# yum -y install sl
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check

Installed:
  sl.x86_64 0:5.02-1.el7                                                                                                                                                                                          

Complete!
[root@shaoxh yum.repos.d]# sl
[root@shaoxh yum.repos.d]# sl -a
```

卸载：

```
[root@shaoxh ~]# yum -y remove httpd
	

```

查看yum历史和日志

```
[root@shaoxh ~]# yum history
Loaded plugins: fastestmirror
ID     | Login user               | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
     8 | root <root>              | 2020-08-27 02:57 | Erase          |    1   
     7 | root <root>              | 2020-08-27 02:28 | Install        |    1   
     6 | root <root>              | 2020-08-27 02:27 | Install        |    1   
     5 | root <root>              | 2020-08-27 02:18 | Install        |   51   
     4 | root <root>              | 2020-08-27 02:12 | Install        |    5   
     3 | root <root>              | 2020-08-27 00:30 | Install        |   31  <
     2 | root <root>              | 2020-08-27 00:16 | Install        |    3 > 
     1 | System <unset>           | 2020-06-26 22:02 | Install        |  311   
history list
[root@shaoxh ~]# cat /var/log/yum.log 

```

根据id号来undo 撤销

```
[root@shaoxh ~]# yum -y history undo 9 
Loaded plugins: fastestmirror
Undoing transaction 9, from Thu Aug 27 03:04:07 2020
    Install httpd-2.4.6-67.el7.centos.x86_64 @base
Resolving Dependencies
--> Running transaction check
---> Package httpd.x86_64 0:2.4.6-67.el7.centos will be erased
--> Finished Dependency Resolution

Dependencies Resolved

==================================================================================================================================================================================================================
 Package                                        Arch                                            Version                                                      Repository                                      Size
==================================================================================================================================================================================================================
Removing:
 httpd                                          x86_64                                          2.4.6-67.el7.centos                                          @base                                          9.4 M

Transaction Summary
==================================================================================================================================================================================================================
Remove  1 Package

Installed size: 9.4 M
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : httpd-2.4.6-67.el7.centos.x86_64                                                                                                                                                               1/1 
  Verifying  : httpd-2.4.6-67.el7.centos.x86_64                                                                                                                                                               1/1 

Removed:
  httpd.x86_64 0:2.4.6-67.el7.centos                                                                                                                                                                              

Complete!

```

检查是否卸载成功

```
root@shaoxh ~]# rpm -q httpd
package httpd is not installed

```

redo重做

```
[root@shaoxh ~]# yum -y history redo 9
Loaded plugins: fastestmirror
Repeating transaction 9, from Thu Aug 27 03:04:07 2020
    Install httpd-2.4.6-67.el7.centos.x86_64 @base
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package httpd.x86_64 0:2.4.6-67.el7.centos will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==================================================================================================================================================================================================================
 Package                                        Arch                                            Version                                                       Repository                                     Size
==================================================================================================================================================================================================================
Installing:
 httpd                                          x86_64                                          2.4.6-67.el7.centos                                           base                                          2.7 M

Transaction Summary
==================================================================================================================================================================================================================
Install  1 Package

Total download size: 2.7 M
Installed size: 9.4 M
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : httpd-2.4.6-67.el7.centos.x86_64                                                                                                                                                               1/1 
  Verifying  : httpd-2.4.6-67.el7.centos.x86_64                                                                                                                                                               1/1 

Installed:
  httpd.x86_64 0:2.4.6-67.el7.centos                                                                                                                                                                              

Complete!

```



配置多个url源

```
[root@shaoxh yum.repos.d]# cat centos.repo
[base]
name=shaoxh
baseurl=file:///misc/cd
	https://mirrors.aliyun.com/centos/7/os/x86_64/
enabled=1
gpgcheck=0

```

**yum源**

​	阿里云repo文件

​	       http://mirrors.aliyun.com/repo

​	CentOS系统的yum源

​			阿里云：https://mirrors.aliyun.com/centos/$releasever/os/x86_64/

​			清华大学：https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/x86_64/

EPEL的yum源

​			阿里云：https://mirrors.aliyun.com/epel/$releaserver/x86_64

阿里巴巴开源软件

​			https://opsx.alibaba.com



**yum-config-manager**

生成172.16.0.1_cobbler_ks_mirror_CentOS-X-x86_64_.repo 

 yum-config-manager   --add-repo= http://172.16.0.1/cobbler/ks_mirror/7/ 

yum-config-manager --disable “仓库名"  禁用仓库 

yum-config-manager --enable  “仓库名” 启用仓库 



**yum命令**

yum命令的用法

​	yum [options] [command] [package...]

显示仓库列表：

​	yum repolist [all|enabled|disabled]

显示程序包

​	yum	list

   yum list [all | glob_exp1] [glob_exp2] [...] 

​     yum list {available|installed|updates} [glob_exp1] [...] 



​    yum  provides 查询属于哪个包

```
[root@bogon yum.repos.d]# yum provides /etc/mime.types
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
mailcap-2.1.41-2.el7.noarch : Helper application and MIME type associations for file types
Repo        : base
Matched from:
Filename    : /etc/mime.types



mailcap-2.1.41-2.el7.noarch : Helper application and MIME type associations for file types
Repo        : @base
Matched from:
Filename    : /etc/mime.types


```

yum list installed: 查询已经安装的包@

yum list all  :查询全部的包

yum list available：没有安装的包

yum repolist all : 显示所有的仓库

reinstall重新安装：

```
[root@bogon yum.repos.d]# yum reinstall tree
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com

```

升级程序包：

​	yum update [package1]

​     yum downgrade package1 [package2] [...] (降级) 

检查可用升级

​     yum check-update 

卸载程序包：

​       yum remove | erase package1 [package2] [...] 

查看程序包information： 

​		yum info [...] 

查看指定的特性(可以是某文件)是由哪个程序包所提供： 

​	   yum provides | whatprovides feature1 [feature2] [...] 

清理本地缓存： 

​	   清除/var/cache/yum/$basearch/$releasever缓存 

​		yum clean [ packages | metadata | expire-cache | rpmdb | plugins |all

构建缓存： 

​         yum makecache 

搜索：

​           yum search string1 [string2] [...] 

查看指定包所依赖的capabilities： 

​		yum deplist package1 [package2] [...] 

查看yum事务历史： 

​		yum history [info|list|packages-list|packages-info|summary|addon-info|redo|undo| rollback|new|sync|stats] 

​		yum history 

​		yum history info 6 

​		yum history undo 6 

日志 ：/var/log/yum.log 

安装及升级本地程序包： 

yum localinstall rpmfile1 [rpmfile2] [...] 

 (用install替代) 

yum localupdate rpmfile1 [rpmfile2] [...] 

 (用update替代) 

包组管理的相关命令： 

yum groupinstall group1 [group2] [...] 

yum groupupdate group1 [group2] [...] 

yum grouplist [hidden] [groupwildcard] [...] 

yum groupremove group1 [group2] [...] 

yum groupinfo group1 [...] 

yum的命令行选项： 

--nogpgcheck：禁止进行gpg check 

-y: 自动回答为“yes” 

-q：静默模式 

--disablerepo=repoidglob：临时禁用此处指定的repo 

--enablerepo=repoidglob：临时启用此处指定的repo 

--noplugins：禁用所有插件 



实验：基于httpd协议的yum源

1.selinux 关闭

2.systemctl stop firewalld 关闭

```
yum install httpd -y
systemctl start httpd
cd /var/www/html
mkdir centos/{6,7}/os/x86_64 -pv
echo '- - -' > /sys/class/scsi_host/host2/scan
mount /dev/sr0 centos/7/os/x86_64
mount /dev/sr1 centos/6/os/x86_64
```

定义多个源地址文件：

```
[root@bogon html]# cat yum.txt 
https://mirrors.aliyun.com/centos/7/
http://192.168.1.32/centos/7/os/x86_64/

```

配置客户端仓库：

```
[root@shaoxh yum.repos.d]# cat test.repo 
[base]
name= base yum
#baseurl=http://192.168.1.32/centos/7/os/x86_64/
mirrorlist=http://192.168.1.32/yum.txt
gpgcheck=0

```

yum groups

```
[root@bogon html]# yum groups 
info     install  list     remove   summary  

```

yum group info

```
[root@bogon html]# yum groups info "Development Tools"
Loaded plugins: fastestmirror, langpacks
.....
   +swig
   +systemtap
 Optional Packages:
   ElectricFence
   ant
   babel
   bzr
   ccache
.......
```

yum groupinstall

```
[root@bogon html]# yum -y groupinstall "Development Tools"

```

**系统光盘仓库**

​     系统安装光盘作为本地yum仓库

​	1.挂载光盘至某目录，例如/mnt/cdrom

​		mount /dev/sr0 /mnt/cdrom

​    2.创建配置文件

​		[centos 7]

​		name=

​		baseurl=

​		gpgcheck=

​		enabled=

创建yum仓库：

​	createrepo [options]  <directory>

```
[root@bogon html]# mv /root/*.rpm zlib/
[root@bogon html]# cd zlib/
[root@bogon zlib]# ls
tree-1.6.0-10.el7.x86_64.rpm  zlib-1.2.7-17.el7.x86_64.rpm      zlib-devel-1.2.7-17.el7.x86_64.rpm  zlib-static-1.2.7-17.el7.x86_64.rpm
zlib-1.2.7-17.el7.i686.rpm    zlib-devel-1.2.7-17.el7.i686.rpm  zlib-static-1.2.7-17.el7.i686.rpm
[root@bogon zlib]# createrepo .
Spawning worker 0 with 7 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
[root@bogon zlib]# ls
repodata                      zlib-1.2.7-17.el7.i686.rpm    zlib-devel-1.2.7-17.el7.i686.rpm    zlib-static-1.2.7-17.el7.i686.rpm
tree-1.6.0-10.el7.x86_64.rpm  zlib-1.2.7-17.el7.x86_64.rpm  zlib-devel-1.2.7-17.el7.x86_64.rpm  zlib-static-1.2.7-17.el7.x86_64.rpm

```

路径就为/var/www/html/zlib

```
[root@shaoxh yum.repos.d]# cat zlib.repo
[zlib]
name=zlib
baseurl=http://192.168.1.32/zlib/
gpgcheck=0
[root@shaoxh yum.repos.d]# yum repolist

base                                                                                                                                                                                           9,591
zlib                                                                                                 zlib                                                                                                       7
repolist: 9,598
[root@shaoxh yum.repos.d]# 

```

**DNF**

DNF 介绍：新一代的RPM软件包管理器。DNF 发行日期是2015年5月11日，DNF 包管 理器采用Python 编写，发行许可为GPL v2，首先出现在Fedora 18 发行版中。在 RHEL 8.0 版本正式取代了 YUM，DNF包管理器克服了YUM包管理器的一些瓶颈，提升 了包括用户体验，内存占用，依赖分析，运行速度等 

安装所需软件包，依赖epel源

配置文件:：/etc/dnf/dnf.conf

仓库文件：/etc/yum.repos.d/*.repo

日志：/var/log/dnf.rpm.log

**DNF使用**

帮助：man dnf

dnf 用法：与yum一致

dnf [options] <command> [<argument>]

dnf --version

dnf repolist

dnf clean all

dnf makecache

dnf list installed

dnf list available

dnf  search nano

dnf history

dnf history undo 1

**程序包编译**

程序包编译安装

Application-VERSION-release.src.rpm --> 安装后，使用rpmbuild命令制作成二进制格式的rpm包，而后在安装

源代码-->预处理-->编译-->汇编--> 链接-->执行

源代码组织格式：

​	多文件：文件中的代码之间，很可能存在跨文件依赖关系

​	C、C++: make 项目管理器

​		configure脚本-->Makefile.in --> Makefile

java:maven

**编译安装**

C语言源代码编译安装三步骤：

1../configure

通过选项传递参数，指定启用特性、安装路径等；执行时会参考用户的指定以及Makefile.in文件生成Makefile

检查依赖到的外部环境，如依赖的软件包

2.make 根据Makefile文件，构建应用程序

3.make install 复制文件到相应路径

开发工具

​		autoconf:生成configure脚本

​		automake:生成Makefile.in

注意:安装前查看INSTALL,README

编译安装tree命令1.8

```
[root@node2 ~]# wget http://mama.indstate.edu/users/ice/tree/src/tree-1.8.0.tgz
--2020-09-03 18:57:06--  http://mama.indstate.edu/users/ice/tree/src/tree-1.8.0.tgz
Resolving mama.indstate.edu (mama.indstate.edu)... 139.102.70.201
Connecting to mama.indstate.edu (mama.indstate.edu)|139.102.70.201|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 50286 (49K) [application/x-gzip]
Saving to: ‘tree-1.8.0.tgz’

100%[========================================================================================================================================================================>] 50,286      35.5KB/s   in 1.4s   

2020-09-03 18:57:07 (35.5 KB/s) - ‘tree-1.8.0.tgz’ saved [50286/50286]

[root@node2 ~]# tar xf tree-1.8.0.tgz 
[root@node2 ~]# ls
anaconda-ks.cfg  initial-setup-ks.cfg  tree-1.8.0  tree-1.8.0.tgz

```

查看Makefile文件内容信息

```
[root@node2 tree-1.8.0]# vim Makefile 
[root@node2 tree-1.8.0]# 
[root@node2 tree-1.8.0]# 
[root@node2 tree-1.8.0]# gcc
bash: gcc: command not found...
[root@node2 tree-1.8.0]# yum -y install gcc

```

修改Makefile文件的默认安装路径

```
vim Makefile
prefix = /apps/tree


```

选择都多进程并行编译

```
[root@node2 tree-1.8.0]# make -j 2
gcc -ggdb -pedantic -Wall -DLINUX -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64 -c -o tree.o tree.c
gcc -ggdb -pedantic -Wall -DLINUX -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64 -c -o unix.o unix.c
tree.c: In function ‘main’:
tree.c:585:86: warning: ISO C90 does not support ‘long long’ [-Wlong-long]
       if (duflag) fprintf(outfile,"%s<size>%lld</size>%s", noindent?"":"    ", (long long int)size, _nl);
                                                                                      ^
tree.c:585:7: warning: ISO C90 does not support the ‘ll’ gnu_printf length modifier [-Wformat=]
       if (duflag) fprintf(outfile,"%s<size>%lld</size>%s", noindent?"":"    ", (long long int)size, _nl);
       ^
tree.c:591:59: warning: ISO C90 does not support ‘long long’ [-Wlong-long]
       if (duflag) fprintf(outfile,",\"size\":%lld", (long long int)size);
                                                           ^
tree.c:591:7: warning: ISO C90 does not support the ‘ll’ gnu_printf length modifier [-Wformat=]
       if (duflag) fprintf(outfile,",\"size\":%lld", (long long int)size);
       ^
tree.c: In function ‘usage’:
tree.c:695:2: warning: string length ‘3106’ is greater than 
```

生成了tree命令

```
[root@node2 tree-1.8.0]# 
[root@node2 tree-1.8.0]# ls
CHANGES  color.o  file.c  hash.c  html.c  INSTALL  json.o   Makefile  strverscmp.c  tree    tree.h  unix.c  xml.c
color.c  doc      file.o  hash.o  html.o  json.c   LICENSE  README    TODO          tree.c  tree.o  unix.o  xml.o
[root@node2 tree-1.8.0]# ls /apps/tree
ls: cannot access /apps/tree: No such file or directory
[root@node2 tree-1.8.0]# make install
install -d /apps/tree/bin
install -d /apps/tree/man/man1
if [ -e tree ]; then \
	install tree /apps/tree/bin/tree; \
fi
install doc/tree.1 /apps/tree/man/man1/tree.1

[root@node2 tree-1.8.0]#  /apps/tree/bin/tree /apps/
/apps/
└── tree
    ├── bin
    │   └── tree
    └── man
        └── man1
            └── tree.1

4 directories, 2 files
[root@node2 tree-1.8.0]# 

```

编译安装 httpd-2.4.25.tar.bz2,

```
[root@node2 ~]# tar xf httpd-2.4.25.tar.bz2 
[root@node2 ~]# cd httpd-2.4.25/
[root@node2 httpd-2.4.25]# ls
ABOUT_APACHE     apache_probes.d  BuildBin.dsp    config.layout  emacs-style  httpd.spec      LAYOUT        LICENSE       NOTICE         README.cmake      srclib
acinclude.m4     ap.d             buildconf       configure      httpd.dep    include         libhttpd.dep  Makefile.in   NWGNUmakefile  README.platforms  support
Apache-apr2.dsw  build            CHANGES         configure.in   httpd.dsp    INSTALL         libhttpd.dsp  Makefile.win  os             ROADMAP           test
Apache.dsw       BuildAll.dsp     CMakeLists.txt  docs           httpd.mak    InstallBin.dsp  libhttpd.mak  modules       README         server            VERSIONING
[root@node2 httpd-2.4.25]# 

```

安装帮助

```
[root@node2 httpd-2.4.25]# vim README
[root@node2 httpd-2.4.25]# 
[root@node2 httpd-2.4.25]# vim INSTALL 
[root@node2 httpd-2.4.25]# 
[root@node2 httpd-2.4.25]# ./configure --help

```

```
[root@node2 httpd-2.4.25]# ./configure --prefix=/apps/httpd24 --sysconfdir=/etc/httpd --enable-ssl --enable-so
checking for chosen layout... Apache
checking for working mkdir -p... yes
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking target system type... x86_64-unknown-linux-gnu
configure: 
configure: Configuring Apache Portable Runtime library...
configure: 
checking for APR... no
configure: error: APR not found.  Please read the documentation.
[root@node2 httpd-2.4.25]# yum search apr
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.bfsu.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirror.bit.edu.cn
================================================================================================ N/S matched: apr ================================================================================================
apr-devel.i686 : APR library development kit
apr-devel.x86_64 : APR library development kit
apr-util-devel.i686 : APR utility library development kit
apr-util-devel.x86_64 : APR utility library development kit
apr-util-ldap.x86_64 : APR utility library LDAP support
apr-util-mysql.x86_64 : APR utility library MySQL DBD driver
apr-util-nss.x86_64 : APR utility library NSS crytpo support
apr-util-odbc.x86_64 : APR utility library ODBC DBD driver
apr-util-openssl.x86_64 : APR utility library OpenSSL crytpo support
apr-util-pgsql.x86_64 : APR utility library PostgreSQL DBD driver
apr-util-sqlite.x86_64 : APR utility library SQLite DBD driver
pcp-pmda-haproxy.x86_64 : Performance Co-Pilot (PCP) metrics for HAProxy
apr.i686 : Apache Portable Runtime library
apr.x86_64 : Apache Portable Runtime library
apr-util.i686 : Apache Portable Runtime Utility library
apr-util.x86_64 : Apache Portable Runtime Utility library
haproxy.x86_64 : TCP/HTTP proxy and load balancer for high availability environments

  Name and summary matches only, use "search all" for everything.
[root@node2 httpd-2.4.25]# yum -y install apr-devel


[root@node2 httpd-2.4.25]# ./configure --prefix=/apps/httpd24 --sysconfdir=/etc/httpd --enable-ssl --enable-so
checking for chosen layout... Apache
checking for working mkdir -p... yes
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking target system type... x86_64-unknown-linux-gnu
configure: 
configure: Configuring Apache Portable Runtime library...
configure: 
checking for APR... yes
  setting CC to "gcc"
  setting CPP to "gcc -E"
  setting CFLAGS to "  -pthread"
  setting CPPFLAGS to " -DLINUX -D_REENTRANT -D_GNU_SOURCE"
  setting LDFLAGS to " "
configure: 
configure: Configuring Apache Portable Runtime Utility library...
configure: 
checking for APR-util... no
configure: error: APR-util not found.  Please read the documentation.
[root@node2 httpd-2.4.25]# yum -y install apr-util-devel
[root@node2 httpd-2.4.25]# yum -y install pcre-devel
[root@node2 httpd-2.4.25]# yum -y install openssl-devel
[root@node2 httpd-2.4.25]# make && make install
```

启动httpd:

```
[root@node2 bin]# /apps/httpd24/bin/apachectl start
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::2093:424d:ee51:2539. Set the 'ServerName' directive globally to suppress this message
[root@node2 bin]# ss -tnl
[root@node2 bin]# iptables -F
[root@node2 bin]# setenforce 0

```

 重启也能够启动服务

```
[root@node2 bin]# ll /etc/rc.d/rc.local 
-rw-r--r--. 1 root root 473 Aug  5  2017 /etc/rc.d/rc.local
[root@node2 bin]# vim /etc/rc.d/rc.local 
                 /apps/httpd24/bin/apachectl start

[root@node2 bin]# chmod +x /etc/rc.d/rc.local 
[root@node2 bin]# 
[root@node2 bin]# cat /etc/rc.d/rc.local

```

```
[root@node2 ~]# curl localhost
<html><body><h1>It works!</h1></body></html>
[root@node2 ~]# 
[root@node2 ~]# 
[root@node2 ~]# cd /apps/httpd24/htdocs/
[root@node2 htdocs]# ls
index.html
[root@node2 htdocs]# vim index.html 
[root@node2 htdocs]# 
[root@node2 htdocs]# curl localhost
<html><body><h1>shaoxhIt works!</h1></body></html>

```

编译安装cmatrix-1.2a.tar.gz

注意,必须先安装

yum -y install ncurses-devel

```
 153  ./configure --prefix=/apps/cmatrix
  154  make 
  155  make install
  /apps/cmatrix/bin/cmatrix  

```

建立一个网站后上传一个脚本,在客户端执行脚本

```
[root@node2 htdocs]# cat test.sh 
#!/bin/bash
hostname
useradd magedu
[root@node2 htdocs]# 

```

客户端远程执行脚本

```
[root@node1 ~]# curl http://192.168.1.56/test.sh|bash

```

































































