# ANSIBLE

## 1.1.1 安装ansible

```
[root@node1 ~] yum -y install epel*
[root@node1 ~] yum -y install ansible

```

## 1.1.2 列出ansible的模块

```
[root@node1 ~]# ansible-doc -l
```

## 1.1.3 查看hostname模块

```
[root@node1 ~]# ansible-doc -s hostname

```

## 1.1.4 查看被ansible管理的主机

```
[root@node1 ~]# ansible all --list-hosts
  hosts (2):
    192.168.0.113
    192.168.0.112

[root@node1 ~]# ansible websrvs --list-hosts
  hosts (1):
    192.168.0.112

```

## 1.1.5 查看被管理的主机是否宕机

```
建议把/etc/ansible/ansible.cfg 文件中的 host_key_checking=False 打开，
一次连接就不会出现检查key ，确认yes no值
[root@node1 ~]# ansible appsrvs -m ping -k
SSH password: 
192.168.0.113 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}

```

## 1.1.6 基于key验证

```
[root@node1 ~]# ssh-keygen 
[root@node1 ~]# ssh-copy-id 192.168.0.112
[root@node1 ~]# ssh-copy-id 192.168.0.113

[root@node1 ~]# ansible all -m ping 

不执行，做检查
[root@node1 ~]# ansible all -m ping -C

```

## 1.1.7 主机可以使用通配符

```
匹配主机的列表
ALL表示所有主机
[root@node1 ~]# ansible all -m ping

通配符*
[root@node1 ~]# ansible "*" -m ping
[root@node1 ~]# ansible "*srvs" -m ping

或关系
[root@node1 ~]# ansible "websrvs:appsrvs" -m ping
[root@node1 ~]# ansible "192.168.0.112:192.168.0.113" -m ping

逻辑与
[root@node1 ~]# ansible "websrvs:&appsrvs" -m ping

逻辑非
[root@node1 ~]# ansible 'websrvs:!appsrvs' -m ping
此条命令必须用单引号

综合逻辑
[root@node1 ~]# ansible 'websrvs:appsrvs:&appsrvs:!ftpsrvs' -m ping

正则表达式
[root@node1 ~]# ansible "~(web|app)srvs" -m ping



```

## 1.1.8 ansible 开启日志功能

```
log_path = /var/log/ansible.log

```

## 1.1.9 commad模块

```
不支持 $ | > 这样的符号
[root@node1 ~]# ansible-doc -s command
[root@node1 ~]# ansible all -m command -a "ls /tmp"
[root@node1 ~]# ansible all -m command -a "chdir=/tmp ls"   #切换目录执行 ls命令
[root@node1 ~]# ansible all -m command -a "creates=/etc/fstab ls /tmp"  #文件存在，不执行ls命令
[root@node1 ~]# ansible all -m command -a "removes=/etc/fstab ls /tmp"  #文件存在，执行ls命令
```

## 1.2.1 shell模块

```
[root@node1 ~]# ansible all -m shell -a "echo $HOSTNAME"

```

## 1.2.2 修改默认模块

```
module_name = shell
[root@node1 ~]# ansible all -a "sed -i 's@SELINUX=enforcing@SELINUX=disabled@' /etc/selinux/config"

```

## 1.2.3 script 模块

```
[root@node1 ~]# chmod +x test.sh 
[root@node1 ~]# ansible all -m script -a '/root/test.sh'

```

## 1.2.4 copy 模块

```
[root@node1 ~]# ansible-doc -s copy
[root@node1 ~]# ansible websrvs -m copy -a 'src=/etc/fstab dest=/tmp'
[root@node1 ~]# ansible websrvs -a 'ls -l /tmp'

[root@node1 ~]# ansible websrvs -m copy -a 'src=/etc/passwd dest=/tmp mode=600 owner=shaoxh group=shaoxh'
[root@node1 ~]# ansible websrvs -a 'ls -l /tmp'

如果存在可以指定备份
[root@node1 ~]# ansible websrvs -m copy -a 'src=/etc/issue dest=/tmp/passwd mode=600 owner=shaoxh group=shaoxh backup=yes'

自定义内容
[root@node1 ~]# ansible websrvs -m copy -a 'content="line1\nline2\nline3" dest=/tmp/txt'
[root@node1 ~]# ansible websrvs -a 'cat  /tmp/txt'
192.168.0.112 | CHANGED | rc=0 >>
line1
line2
line3


```

## 1.2.5 Fetch 模块

```
把远程的文件抓取到本地目录
[root@node1 ~]# ansible websrvs -m fetch -a 'src=/var/log/messages dest=/data'

```

## 1.2.6 file模块

```
[root@node1 log]# ansible-doc -s file
[root@node1 log]# ansible all -a "mkdir /data"
[root@node1 log]# ansible websrvs -m copy -a 'src=/etc/fstab dest=/data'
[root@node1 log]# ansible websrvs -m file -a 'path=/data/fstab owner=shaoxh mode=700'
[root@node1 log]# ansible websrvs -a 'ls -l /data'
192.168.0.112 | CHANGED | rc=0 >>
total 4
-rwx------. 1 shaoxh root 465 Dec  1 02:51 fstab

创建软连接
[root@node1 log]# ansible websrvs -m file -a 'src=/data/fstab path=/data/fstab.link state=link'

创建硬链接
[root@node1 log]# ansible websrvs -m file -a 'src=/data/fstab path=/data/fstab.link2 state=hard'

创建空文件
[root@node1 log]# ansible websrvs -m file -a 'path=/data/f1.txt state=touch'

删除文件
[root@node1 log]# ansible websrvs -m file -a 'path=/data/f1.txt state=absent'

删除目录
[root@node1 log]# ansible websrvs -m file -a 'path=/data/ state=absent'

```

## 1.2.7 unarchive 模块

```
创建一个压缩文件
[root@node1 log]# tar zcvf /data/sysconfig.tar.gz /etc/sysconfig/

把压缩文件在远程主机上解压
[root@node1 log]# ansible websrvs -m unarchive -a 'src=/data/sysconfig.tar.gz dest=/data owner=shaoxh mode=700'

删除远程的/data/etc 目录
[root@node1 log]# ansible websrvs -m file -a "path=/data/etc/ state=absent"

先把压缩文件拷贝远程主机上，在解压
[root@node1 log]# ansible websrvs -m copy -a 'src=/data/sysconfig.tar.gz dest=/data'
[root@node1 log]# ansible websrvs -m unarchive -a 'src=/data/sysconfig.tar.gz dest=/data/ copy=no'

```

## 1.2.8 archive 模块

```
打包远程主机的 /etc/sysconfig 文件夹，保存位置为/data/sysconfig.tar.bz2
[root@node1 ~]# ansible all -m archive -a 'path=/etc/sysconfig dest=/data/sysconfig.tar.bz2 format=bz2 owner=shaoxh mode=0777'

```

## 1.2.9 hostname

```
[root@node1 ~]# ansible websrvs -m hostname -a 'name=centos7-2'

```

## 1.3.1 cron 模块

```
[root@node1 ~]# ansible websrvs -m cron -a "minute=*/5 job='/usr/sbin/ntpdate 0.centos.pool.ntp.org &> /dev/null' name=Synctime"

[root@node1 ~]# ansible websrvs -m shell -a "crontab -l"
192.168.0.112 | CHANGED | rc=0 >>
#Ansible: Synctime
*/5 * * * * /usr/sbin/ntpdate 0.centos.pool.ntp.org &> /dev/null

禁用
[root@node1 ~]# ansible websrvs -m cron -a "minute=*/5 job='/usr/sbin/ntpdate 0.centos.pool.ntp.org &> /dev/null' name=Synctime disabled=true"

删除
[root@node1 ~]# ansible websrvs -m cron -a "minute=*/5 job='/usr/sbin/ntpdate 0.centos.pool.ntp.org &> /dev/null' name=Synctime state=absent"


```

## 1.3.2 yum模块

```
安装httpd
[root@node1 ~]# ansible websrvs -m yum -a 'name=httpd'

卸载httpd
[root@node1 ~]# ansible websrvs -m yum -a 'name=httpd state=absent'

```

## 1.3.3 service 模块

```
启动httpd
[root@node1 ~]# ansible websrvs -m service -a 'name=httpd state=started enabled=yes'

停止httpd
[root@node1 ~]# ansible websrvs -m service -a 'name=httpd state=stopped'

修改httpd端口8080
[root@node1 ~]# ansible websrvs -m shell -a "sed -i 's#^Listen.*#Listen 8080#' /etc/httpd/conf/httpd.conf"

启动服务
[root@node1 ~]# ansible websrvs -m service -a 'name=httpd state=started'

查看端口
[root@node1 ~]# ansible websrvs -a 'ss -tnl'

```

## 1.3.4 user 模块

```
创建mysql用户 指定 系统用户和shell类型 家目录
[root@node1 ~]# ansible websrvs -m user -a "name=mysql system=yes home=/data/mysql shell=/bin/false"

删除mysql并且删除家目录
[root@node1 ~]# ansible websrvs -m user -a "name=mysql state=absent remove=yes"


不创建家目录[root@node1 ~]# ansible websrvs -m user -a "name=mysql system=yes home=/data/mysql shell=/bin/false create_home=no"

```

## 1.3.5 group 模块

```
创建组
[root@node1 ~]# ansible websrvs -m group -a "name=testgroup system=yes"

删除组
[root@node1 ~]# ansible websrvs -m group -a "name=testgroup state=absent"

```

## 1.3.6 galaxy 

```
下载ntp
[root@node1 ~]# ansible-galaxy install geerlingguy.ntp

列出所有已经按照的galaxy
[root@node1 ~]# ansible-galaxy list

删除galaxy
[root@node1 ~]# ansible-galaxy remove geerlingguy.ntp


```

## 1.3.7 创建一个playbook

```
[root@node1 playbook]# cat hello.yml 
- hosts: websrvs
  remote_user: root
  tasks:
    - name: hello world
      command: /usr/bin/wall hello world

[root@node1 ~]# mkdir /data/playbook
[root@node1 ~]# cd /data/playbook
[root@node1 playbook]# vim hello.yml
[root@node1 playbook]# 
[root@node1 playbook]# ansible-playbook hello.yml 

```

## 1.3.8 加密yml文件

```
加密
[root@node1 playbook]# ansible-vault encrypt hello.yml 

查看
[root@node1 playbook]# ansible-vault view hello.yml 
Vault password: 
- hosts: websrvs
  remote_user: root
  tasks:
    - name: hello world
      command: /usr/bin/wall hello world

编辑加密文件
[root@node1 playbook]# ansible-vault edit hello.yml 

修改口令
[root@node1 playbook]# ansible-vault rekey hello.yml 


解密
[root@node1 playbook]# ansible-vault decrypt hello.yml


创建新yml文件
[root@node1 playbook]# ansible-vault create new.yml


```

## 1.3.9 ansible-console

```
root@node1 roles]# ansible-console
Welcome to the ansible console.
Type help or ? to list commands.

root@all (2)[f:5]$ 
root@all (2)[f:5]$ list
192.168.0.113
192.168.0.112
root@all (2)[f:5]$ forks 10
root@all (2)[f:10]$ cd websrvs
root@websrvs (1)[f:10]$ yum name=mariadb-server
root@websrvs (1)[f:10]$ exit

```



## 1.4.1 定义一个httpd安装的yml

```
[root@node1 playbook]# grep 888 httpd.conf 
Listen 888

[root@node1 playbook]# cat httpd.yml 
---
- hosts: websrvs
  remote_user: root

  tasks:
    - name: install httpd
      yum: name=httpd
    - name: copy httpd config
      copy: src=/data/playbook/httpd.conf dest=/etc/httpd/conf/httpd.conf
    - name: start httpd
      service: name=httpd state=restarted enabled=yes
      
[root@node1 playbook]# ansible websrvs -a 'ss -tnl'|grep 888
LISTEN     0      128         :::888                     :::*            

列出哪些主机
[root@node1 playbook]# ansible-playbook --list-hosts httpd.yml

列出哪些任务
[root@node1 playbook]# ansible-playbook --list-tasks httpd.yml


列出标签



```

## 1.4.2 定义一个安装mysql的yml

```
首先需要maridb-server的安装包：
[root@node1 playbook]# ls
install_mysql.yml  mariadb-10.2.25-linux-x86_64.tar.gz  my-huge.cnf

配置文件中需要加入
[root@node1 playbook]# grep datadir my-huge.cnf 
datadir         = /data/mysql


[root@node1 playbook]# cat install_mysql.yml 
---
- hosts: websrvs
  remote_user: root

  tasks:
    - name: create user
      user: name=mysql system=yes home=/data/mysql create_home=no shell=/sbin/nologin
    - name: unarchive 
      unarchive: 
        src: /data/playbook/mariadb-10.2.25-linux-x86_64.tar.gz
        dest: /usr/local/
        owner: root
        group: root
    - name: mysql link
      file: 
        src: /usr/local/mariadb-10.2.25-linux-x86_64 
        dest: /usr/local/mysql 
        state: link
    - name: mysql datadir
      file: 
        path: /data/mysql 
        state: directory
        owner: mysql 
        group: mysql
    - name: mysql database
      shell: chdir=/usr/local/mysql/ scripts/mysql_install_db --datadir=/data/mysql --user=mysql
    - name: path var
      copy: 
        content: 'PATH=/usr/local/mysql/bin:$PATH' 
        dest: /etc/profile.d/mysql.sh
    - name: mysql config
      copy: 
        src: /data/playbook/my-huge.cnf 
        dest: /etc/my.cnf
    - name: service file
      copy: 
        src: /usr/local/mysql/support-files/mysql.server 
        dest: /etc/init.d/mysqld 
        mode: 755 
    - name: start service
      shell: /etc/init.d/mysqld start 
      
 

```

## 1.4.3 定义一个handlers 

```
需要一个httpd.conf 配置文件
[root@node1 playbook]# grep 9527 httpd.conf
Listen 9527


[root@node1 playbook]# cat httpd.yml 
- hosts: websrvs
  remote_user: root

  tasks:
    - name: install httpd
      yum:
        name: httpd
    - name: copy config
      copy:
        src: /data/playbook/httpd.conf
        dest: /etc/httpd/conf/
      notify: restart httpd
    - name: start httpd service
      service:
        name: httpd
        state: started
        enabled: yes
  handlers:
    - name: restart httpd
      service:
        name: httpd
        state: restarted 

```

## 1.4.4 playbook中tags使用

```
[root@node1 playbook]# cat httpd.yml 
- hosts: websrvs
  remote_user: root

  tasks:
    - name: install httpd
      yum:
        name: httpd
    - name: copy config
      tags: conf
      copy:
        src: /data/playbook/httpd.conf
        dest: /etc/httpd/conf/
      notify: restart httpd
    - name: start httpd service
      tags: service
      service:
        name: httpd
        state: started
        enabled: yes
  handlers:
    - name: restart httpd
      service:
        name: httpd
        state: restarted 

```

## 1.4.5 ansible变量

```
[root@node1 playbook]# ansible websrvs -m setup 
[root@node1 playbook]# ansible websrvs -m setup -a 'filter=ansible_fqdn'
[root@node1 playbook]# ansible websrvs -m setup -a 'filter=ansible_*total*'

```

### 1.4.6 取出内存信息

```
[root@node1 playbook]# ansible websrvs -m setup -a 'filter=ansible_memtotal_mb'
192.168.0.112 | SUCCESS => {
    "ansible_facts": {
        "ansible_memtotal_mb": 976, 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}

```

### 1.4.7 取出版本信息

```
[root@node1 playbook]# ansible websrvs -m setup -a 'filter=ansible_*version*'
192.168.0.112 | SUCCESS => {
    "ansible_facts": {
        "ansible_bios_version": "6.00", 
        "ansible_distribution_major_version": "7", 
        "ansible_distribution_version": "7.4", 
        "ansible_kernel_version": "#1 SMP Tue Aug 22 21:09:27 UTC 2017", 
        "ansible_product_version": "None", 
        "ansible_python_version": "2.7.5", 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}

```

## 1.4.8 使用变量创建一个文件（ 使用的setup变量）

```
[root@node1 playbook]# cat vars.yml 
---
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: create file
      tags: file
      file: 
        name: /data/{{ansible_fqdn}}.log
        state: touch

```

## 1.4.9 使用主机清单定义变量

```
[root@node1 ~]# cat /etc/ansible/hosts
[websrvs]
192.168.0.112 hostname=node2
[appsrvs]
192.168.0.113

[root@node1 playbook]# cat vars.yml
---
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: create file
      tags: file
      file: 
        name: /data/{{hostname}}.log
        state: touch

```

### 1.5.1 主机清单定义共用变量

```
[root@node1 playbook]# cat /etc/ansible/hosts 
[websrvs]
192.168.0.112 hostname=node2
192.168.0.113 hostname=node3

[websrvs:vars]
suf=txt


[appsrvs]
192.168.0.113

[root@node1 playbook]# cat vars.yml 
---
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: create file
      tags: file
      file: 
        name: /data/{{hostname}}.{{suf}}
        state: touch

```

## 1.5.2 命令行指定变量 优先级最高

```
[root@node1 playbook]# ansible-playbook -e hostname=test -e suf=log vars.yml

```

## 1.5.3 在playbook中定义

```
[root@node1 playbook]# cat vars.yml 
---
- hosts: websrvs
  remote_user: root
  vars:
    - hostname: testfile
    - suf: html
  tasks:
    - name: create file
      tags: file
      file: 
        name: /data/{{hostname}}.{{suf}}
        state: touch

```

## 1.5.4 特定的yml文件定义变量

```
[root@node1 playbook]# cat vars.yml 
hostname: testnode
suf: yml
[root@node1 playbook]# cat vars1.yml 
---
- hosts: websrvs
  remote_user: root
  vars_files: vars.yml

  tasks:
    - name: create file
      tags: file
      file:
        name: /data/{{hostname}}.{{suf}} 
        state: touch

```

## 1.5.5 template 模板

```
模板目录要和yml文件平级 命名为j2 结尾
[root@node1 playbook]# mkdir templates
[root@node1 playbook]# cp httpd.conf templates/httpd.conf.j2

修改模板文件的端口
[root@node1 playbook]# vim templates/httpd.conf.j2
Listen {{ httpd_port }}

主机清单定义变量
[root@node1 playbook]# cat /etc/ansible/hosts
[websrvs]
192.168.0.112 httpd_port=8112
192.168.0.113 httpd_port=8113

httpd.yml 文件
[root@node1 playbook]# cat httpd.yml 
- hosts: websrvs
  remote_user: root

  tasks:
    - name: install httpd
      yum:
        name: httpd
    - name: copy config
      tags: conf
      template:
        src: /data/playbook/templates/httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf
      notify: restart httpd
    - name: start httpd service
      tags: service
      service:
        name: httpd
        state: started
        enabled: yes
  handlers:
    - name: restart httpd
      service:
        name: httpd
        state: restarted 

检查语法yml语法：
[root@node1 playbook]# ansible-playbook  -C httpd.yml 
[root@node1 playbook]# ansible-playbook   httpd.yml 
[root@node1 playbook]# ansible websrvs -m shell -a "ss -tnl" |grep 811
LISTEN     0      128         :::8113                    :::*                  
LISTEN     0      128         :::8112                    :::*        

```

### 1.5.6 httpd.conf.j2 模板文件支持运算符号

```
Listen {{ httpd_port+100 }}
[root@node1 playbook]# ansible-playbook   httpd.yml 
[root@node1 playbook]# ansible websrvs -m shell -a "ss -tnl" |grep 82
LISTEN     0      128         :::8213                    :::*                  
LISTEN     0      128         :::8212                    :::*         
```

## 1.5.7 安装nginx使用模板文件

```
查看cpus个数
[root@node1 playbook]# ansible all -m setup  |grep cpu
        "ansible_processor_vcpus": 4, 
        "ansible_processor_vcpus": 4, 

都需要配置epel源
ansible本机需要事先准备好nginx模板文件
[root@node1 playbook]# ls /data/playbook/templates/nginx.conf.j2 
/data/playbook/templates/nginx.conf.j2

把模板文件的启动进程修改为anisble的变量
[root@node1 playbook]# cat /data/playbook/templates/nginx.conf.j2
......
worker_processes {{ansible_processor_vcpus}};
......

定义nginx.yml 文件
[root@node1 playbook]# cat nginx.yml 
---
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: install nginx
      yum: 
        name: nginx
    - name: cp nginx config
      template: 
        src: /data/playbook/templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx
    - name: start nginx service
      service: 
        name: nginx
        state: started
        enabled: yes
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted 

执行playbook 
[root@node1 playbook]# ansible-playbook nginx.yml 
[root@node1 playbook]# ansible websrvs -m shell -a "ps -ef |grep ^nginx| grep -v grep"
192.168.0.112 | CHANGED | rc=0 >>
nginx     82729  82728  0 10:25 ?        00:00:00 nginx: worker process
nginx     82730  82728  0 10:25 ?        00:00:00 nginx: worker process
nginx     82731  82728  0 10:25 ?        00:00:00 nginx: worker process
nginx     82732  82728  0 10:25 ?        00:00:00 nginx: worker process


```

## 1.5.8 使用when 

```
先取出变量
ansible_distribution_major_version
[root@node1 templates]# ansible websrvs -m setup -a "filter=*version*"
        "ansible_distribution_major_version": "7", 
        "ansible_distribution_major_version": "7", 
        "ansible_distribution_major_version": "6",

准备好httpd6.conf httpd7.conf文件，
[root@node5 ~]# cat /etc/redhat-release 
CentOS release 6.7 (Final)
[root@node5 ~]# scp /etc/httpd/conf/httpd.conf 192.168.0.111:/root/httpd6.conf

配置文件监听端口修改成变量
[root@node1 ~]# mv httpd6.conf /data/playbook/templates/httpd6.conf.j2 
Listen {{httpd_port}}   

centos7 的配置文件
[root@node1 ~]# mv httpd7.conf /data/playbook/templates/httpd7.conf.j2 
Listen {{httpd_port}} 

主机清单里定义变量
[root@node1 ~]# cat /etc/ansible/hosts
[websrvs]
192.168.0.112 httpd_port=7777 
192.168.0.113 httpd_port=7777
192.168.0.6   httpd_port=6666


定义httpd的yml文件
[root@node1 playbook]# cat httpd.yml 
- hosts: websrvs
  remote_user: root

  tasks:
    - name: install httpd
      yum:
        name: httpd
    - name: copy config
      tags: conf
      template:
        src: /data/playbook/templates/httpd6.conf.j2
        dest: /etc/httpd/conf/httpd.conf
      notify: restart httpd
      when: ansible_distribution_major_version=="6"
    - name: copy config
      tags: conf
      template:
        src: /data/playbook/templates/httpd7.conf.j2
        dest: /etc/httpd/conf/httpd.conf
      notify: restart httpd
      when: ansible_distribution_major_version=="7"
    - name: start httpd service
      tags: service
      service:
        name: httpd
        state: started
        enabled: yes
  handlers:
    - name: restart httpd
      service:
        name: httpd
        state: restarted 

```

## 1.5.9 with_items 迭代

```
创建用户
[root@node1 playbook]# cat items.yml 
---
- hosts: websrvs
  remote_user: root
  
  tasks: 
    - name: create user
      user: 
        name: "{{item}}"
      with_items:
        - tom
        - alice
        - jack
        - rose

删除用户

[root@node1 playbook]# cat items.yml 
---
- hosts: websrvs
  remote_user: root
  
  tasks: 
    - name: delete user
      user: 
        name: "{{item}}"
        state: absent
        remove: yes
      with_items:
        - tom
        - alice
        - jack
        - rose

```

## 1.6.1 迭代嵌套子变量

```
[root@node1 playbook]# cat item2.yml
---
- hosts: websrvs
  remote_user: root
  tasks:
    - name: add some groups
      group: 
        name: "{{ item }}" 
        state: present
      with_items:
        - group1 
        - group2
        - group3
    - name: add some users
      user: 
        name: "{{ item.name }}" 
        group: "{{ item.group }}"
        state: present
      with_items:
        - { name: "user1", group: "group1" }
        - { name: "user2", group: "group2" }
        - { name: "user3", group: "group3" }

```

## 1.6.2 for循环模板

```
[root@node1 playbook]# cat for1.yml 
---
- hosts: websrvs
  remote_user: root
  vars:
    ports:
      - 81
      - 82
      - 83
  tasks:
    - name: config
      template: src=server.conf.j2 dest=/data/server.conf
[root@node1 playbook]# cat templates/server.conf.j2 
{%for port in ports %}
server {
    listen {{ port}}
}
{%endfor%}

```

## 1.6.2 for 循环 

```
[root@node1 playbook]# cat for2.yml 
---
- hosts: websrvs
  remote_user: root
  vars:
    ports:
      - listen_port: 81
      - listen_port: 82
      - listen_port: 83
  tasks:
    - name: config
      template: src=server2.conf.j2 dest=/data/server2.conf
[root@node1 playbook]# 
[root@node1 playbook]# 
[root@node1 playbook]# cat templates/server2.conf.j2 
{%for port in ports %}
server {
    listen {{ port.listen_port}}
}
{%endfor%}

```

## 1.6.3 for 循环

```
[root@node1 playbook]# cat for3.yml 
---
- hosts: websrvs
  remote_user: root
  vars:
    ports:
      - web1:
        listen_port: 81
        name: web1.magedu.com
        dir: /data/web1
      - web2:
        listen_port: 82
        name: web2.magedu.com
        dir: /data/web1
      - web3:
        listen_port: 83
        name: web3.magedu.com
        dir: /data/web1
  tasks:
    - name: config
      template: src=server3.conf.j2 dest=/data/server3.conf
[root@node1 playbook]# cat templates/server3.conf.j2 
{%for port in ports %}
server {
    listen {{ port.listen_port}}
    server_name {{ port.name }}
    root {{ port.dir }}
}
{%endfor%}

```

## 1.6.4 加入判断条件

```
[root@node1 playbook]# cat for4.yml 
---
- hosts: websrvs
  remote_user: root
  vars:
    ports:
      - web1:
        listen_port: 81
        dir: /data/web1
      - web2:
        listen_port: 82
        name: web2.magedu.com
        dir: /data/web1
      - web3:
        listen_port: 83
        dir: /data/web1
  tasks:
    - name: config
      template: src=server4.conf.j2 dest=/data/server4.conf


[root@node1 playbook]# cat templates/server4.conf.j2 
{%for port in ports %}
server {
    listen {{ port.listen_port}}
{%if port.name is defined %}
    server_name {{ port.name }}
{%endif%}
    root {{ port.dir }}
}
{%endfor%}

```

##        1.6.5 roles

```
创建需要的roles 目录
[root@playbook~]# mkdir roles/{nginx,mysql}/{tasks,files} -p
[root@playbook~]# cd roles/nginx/tasks/

创建任务文件
[root@playbook~]# touch user.yml install.yml config.yml service.yml

[root@node1 nginx]# tree 
.
├── files
│   └── nginx.conf
└── tasks
    ├── config.yml
    ├── install.yml
    ├── main.yml
    ├── service.yml
    └── user.yml

配置roles文件

[root@node1 nginx]# cat tasks/user.yml 
- name: create user
  user: 
    name: nginx
    shell: /sbin/nologin
    system: yes
    create_home: no
[root@node1 nginx]# cat tasks/install.yml 
- name: install
  yum: 
    name: nginx
[root@node1 nginx]# cat tasks/config.yml 
- name: config
  copy: 
    src: nginx.conf 
    dest: /etc/nginx/
[root@node1 nginx]# cat tasks/service.yml 
- name: service
  service:
    name: nginx
    state: started
    enabled: yes
[root@node1 nginx]# cat tasks/main.yml 
- include: user.yml 
- include: install.yml
- include: config.yml
- include: service.yml

进入roles 同级别目录调用yml文件
[root@node1 nginx]# cd /data/playbook/
[root@node1 playbook]# cat nginx_roles.yml 
- hosts: websrvs
  remote_user: root
  
  roles:
    - role: nginx

[root@node1 playbook]# ansible-playbook  nginx_roles.yml 


```

## 1.6.6 定义handlers 

```
[root@node1 nginx]# mkdir handlers
[root@node1 nginx]# cat handlers/main.yml 
- name: restart service
  service:
    name: nginx
    state: restarted

调用notify
[root@node1 nginx]# cat /data/playbook/roles/nginx/tasks/config.yml 
- name: config
  copy: 
    src: nginx.conf 
    dest: /etc/nginx/
  notify: restart service

修改端口为8080
执行playbook
[root@node1 playbook]# ansible-playbook  nginx_roles.yml 

```

## 1.6.7 自定义nginx主页

```
[root@node1 ~]# echo "<h1>welcome to magedu</h1>" > /data/playbook/roles/nginx/files/index.html

拷贝主页面到远程主机
[root@node1 playbook]# cat roles/nginx/tasks/data.yml 
- name: data file
  copy:
    src: index.html
    dest: /usr/share/nginx/html/

引入data.yml文件
[root@node1 playbook]# cat roles/nginx/tasks/main.yml 
- include: user.yml 
- include: install.yml
- include: config.yml
- include: data.yml
- include: service.yml

[root@node1 playbook]# ansible-playbook nginx_roles.yml 

```

## 1.6.8 apache

```
创建apache 目录
[root@node1 roles]# mkdir httpd
[root@node1 roles]# mkdir httpd/{tasks,files}
[root@node1 tasks]# touch install.yml config.yml data.yml service.yml
[root@node1 tasks]# ll
total 0
-rw-r--r--. 1 root root 0 Dec 13 21:25 config.yml
-rw-r--r--. 1 root root 0 Dec 13 21:25 data.yml
-rw-r--r--. 1 root root 0 Dec 13 21:25 install.yml
-rw-r--r--. 1 root root 0 Dec 13 21:25 service.yml

[root@node1 tasks]# ls > main.yml

定义任务
[root@node1 tasks]# cat install.yml 
- name: install
  yum:
    name: httpd

[root@node1 tasks]# cat config.yml 
- name: config file
  template:
    src: httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf

创建模板的目录
[root@node1 tasks]# mkdir /data/playbook/roles/httpd/templates
[root@node1 templates]# cp /etc/httpd/conf/httpd.conf httpd.conf.j2
修改模板文件里面的运行程序的用户和组
User {{username}}
Group {{groupname}}

创建变量的目录
[root@node1 httpd]# mkdir vars
[root@node1 httpd]# vim vars/main.yml
[root@node1 httpd]# 
[root@node1 httpd]# cat vars/main.yml
username: daemon
groupname: daemon

数据目录的内容
[root@node1 tasks]# cat data.yml 
- name: data file
  copy:
    src: /data/playbook/roles/nginx/files/index.html
    dest: /var/www/html/

启动服务的yml
[root@node1 tasks]# cat service.yml
- name: service
  service:
    name: httpd
    state: started
    enabled: yes


main 配置文件
[root@node1 tasks]# cat main.yml
- include: install.yml
- include: config.yml
- include: data.yml
- include: service.yml

httpd yml文件
[root@node1 playbook]# cat httpd_role.yml 
- hosts: appsrvs
  remote_user: root
  
  roles:
    - httpd

执行yml文件
[root@node1 playbook]# ansible-playbook httpd_role.yml


```

## 1.6.9 mariadb 的yml配置

```
创建mysql 任务的yml文件
[root@node1 tasks]# touch user.yml unarchive.yml link.yml datadir.yml database.yml var.yml script.yml service_file.yml start_service.yml

创建用户
[root@node1 tasks]# cat user.yml 
- name: create user
  user: name=mysql system=yes home=/data/mysql create_home=no shell=/sbin/nologin

解压缩
[root@node1 tasks]# cat unarchive.yml 
- name: unarchive 
  unarchive: 
    src: /data/playbook/mariadb-10.2.25-linux-x86_64.tar.gz
    dest: /usr/local/
    owner: root
    group: root

创建软链接
[root@node1 tasks]# cat link.yml 
- name: mysql link
  file: 
    src: /usr/local/mariadb-10.2.25-linux-x86_64 
    dest: /usr/local/mysql 
    state: link

数据安装目录
[root@node1 tasks]# cat datadir.yml
- name: mysql datadir
  file: 
    path: /data/mysql 
    state: directory
    owner: mysql 
    group: mysql

数据目录
[root@node1 tasks]# cat database.yml 
- name: mysql database
  shell: chdir=/usr/local/mysql/ scripts/mysql_install_db --datadir=/data/mysql --user=mysql

变量
[root@node1 tasks]# cat var.yml
- name: path var
  copy: 
    content: 'PATH=/usr/local/mysql/bin:$PATH' 
    dest: /etc/profile.d/mysql.sh

配置文件
[root@node1 tasks]# cat script.yml
- name: mysql config
  copy: 
    src: /data/playbook/my-huge.cnf 
    dest: /etc/my.cnf


启动脚本加上执行权限
[root@node1 tasks]# cat service_file.yml 
- name: service file
  copy: 
    src: /usr/local/mysql/support-files/mysql.server 
    dest: /etc/init.d/mysqld 
    mode: 755 

启动服务
[root@node1 tasks]# cat start_service.yml
- name: start service
  shell: /etc/init.d/mysqld start 

生成全局main.yml
[root@node1 tasks]# cat main.yml
- include: user.yml
- include: unarchive.yml
- include: link.yml
- include: datadir.yml
- include: database.yml
- include: var.yml
- include: script.yml
- include: service_file.yml
- include: start_service.yml

运行yml文件
[root@node1 playbook]# cat mysql_rols.yml 
- hosts: appsrvs
  remote_user: root
  
  roles:
    - mysql

[root@node1 playbook]# ansible-playbook mysql_rols.yml


```



## 1.7.1  roles 基于条件测试

```
[root@node1 playbook]# cat httpd_nginx.yml 
- hosts: websrvs
  remote_user: root
  
  roles: 
    - {role: httpd, when: ansible_distribution_major_version== "6"} 
    - {role: nginx, when: ansible_distribution_major_version== "7"} 

```

 



























































