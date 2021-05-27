# git+Jenkins

## 1.1 安装依赖包

```
 yum install -y curl policycoreutils-python openssh-server perl
 systemctl enable sshd
 systemctl start sshd
 firewall-cmd --permanent --add-service=http
 firewall-cmd --permanent --add-service=https
 systemctl reload firewalld
 yum install postfix
 systemctl enable postfix
 systemctl start postfix

```

## 1.2 下载rpm包

```
gitlab-ce-11.11.5-ce.0.el7.x86_64.rpm
rpm 包国内下载地址：https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/
```

## 1.3 gitlab配置使用

```
vim /etc/gitlab/gitlab.rb
external_url 'http://192.168.120.71'             主机ip或者完全主机名
gitlab_rails['smtp_enable'] = true               开启                
gitlab_rails['smtp_address'] = "smtp.qq.com"       
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "958678109@qq.com"
gitlab_rails['smtp_password'] = "oezypuhvswulbfbe"
gitlab_rails['smtp_domain'] = "qq.com"
gitlab_rails['smtp_authentication'] = :login
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = "958678109@qq.com"
user["git_user_email"] = "958678109@qq.com"

```

## 1.4 初始化服务

```
#gitlab-ctl reconfigure  
```

## 1.5 注册密码

```
shaoxh8086
用户名
root
密码
shaoxh8086
```

## 1.6 创建group

```
点击🔧 --》 New group --> group name (linux36) ....--->create group
点击🔧 --》 New user --> name (user1) -->username (user1) -->Email (9586781109@qq.com)...---> create user
```

## 1.7 重置密码

```
点击用户 --》 右上角 edit --》 password
退出登录--》点击forgotyour passwd？
```

## 1.8 添加用户到组

```
点击 group -->add user to the group
user1
owener
add users to group


user1 登录 --》 点击groups--》your groups -->linux36 --> New project --》Project name（web1）

```

## 1.9 添加文档

```
add README --》master （index.html)
linux web1 v1
Commit message  Add index.html v1
commit changes
```

## 2.1 克隆文件

```
projects --》 your projects --》 linux—web1 --》点击clone
```

## 2.2 客户端克隆web1文件

```
[root@client ~]# git clone http://192.168.120.128/linux36/web1.git
Cloning into 'web1'...
Username for 'http://192.168.120.128': user1
Password for 'http://user1@192.168.120.128':
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
[root@client ~]# ll
total 0
drwxr-xr-x. 3 root root 36 Mar 23 03:06 web1
[root@client ~]# cd web1/
[root@client web1]# ls
index.html

```

## 2.3 github网站创建项目

```
右上角--》点击➕--》New repository
Repository name
LINUX36
Create repository

点击主页最左面shaoxianheng\LINUX36--》Projects --》Create a project 
Project board name (web1)

creating a new file 

[root@client ~]# git clone https://github.com/shaoxianheng/LINUX36.git
Cloning into 'LINUX36'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
[root@client ~]# ls
LINUX36
[root@client ~]# cat LINUX36/
.git/       index.html
[root@client ~]# cat LINUX36/index.html
患难见真情
[root@client ~]#

修改一下文件内容
[root@client LINUX36]# git push git@github.com:shaoxianheng/LINUX36.git


```

