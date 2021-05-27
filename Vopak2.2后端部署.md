# Vopak2.2后端部署

## 	环境准备	

### 系统版本

```
执行如下命令
   lsb_release -a
```

```
显示结果
root@shaoxh:~# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.7 LTS
Release:	16.04
Codename:	xenial

```

### 更新源

```
 apt update -y

```

### 安装openssh-server 

```
apt -y install openssh-server vim 
```

### root 账号远程登录

```
vim /etc/ssh/sshd_config 
修改一行
......
PermitRootLogin yes
.......

重启服务
systemctl restart sshd
```

