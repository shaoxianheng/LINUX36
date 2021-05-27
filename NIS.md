NIS

http://cn.linux.vbird.org/linux_server/0430nis/0430nis-centos4.php#whatisnis

```
#server端安装
yum -y install rpcbind yp-tools ypbind ypserv

#设定NIS的域名
nisdomainname vbirdnis
echo NISDOMAIN=vbirdnis > /etc/sysconfig/network

#设定NIS配置访问权限
cat >> /etc/ypserv.conf << EOF
192.168.0.0/24 : * : * : none
192.168.190.0/24 : * : * : none
*                : * : * :deny
EOF

#设定主机名与建立信任群组
cat >> /etc/hosts << EOF
192.168.0.111 master.vbirdnis
192.168.0.112 slave.vbirdnis
192.168.0.113 client.vbirdnis
EOF

touch /etc/netgroup

#启动所有相关的服务
systemctl restart ypserv yppasswdd rpcbind

#rpcinfo 来检查
rpcinfo -p localhost
rpcinfo -u localhost ypserv

```

