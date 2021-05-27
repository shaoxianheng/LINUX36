# Docker

## 什么是Docker

### Docker 开源项目背景

​	docker 是基于Go语言实现的开源容器项目。它诞生于2013年年初，最初发起者是dotCloud公司。Docker自开源后受到业界广泛的关注和参与，目前已有80多个相关开源组件项目（包括Containerd、Moby、Swarm等），形成了围绕Docker容器的生态体系。

​		DockerCloud公司也随之快速发展壮大，在2013年年底直接改名为Docker Inc,并专注于Docker相关技术和产品的开发，目前已成为全球最大的Docker容器服务提供商。官网为docker.com。



## 下载一个busybox镜像：

```
docker pull busybox
```

## 查看镜像列表

```
docker images
```

## 启动usybox的容器

```
docker run -d busybox
```

## 使用exec连接容器

```0
docker exec -it "ID" sh
mkdir -p /data/www/html
vi /data/www/html/index.html
/bin/httpd -f -h /data/www/html
```

## 宿主机访问容器80端口

```
curl 172.17.0.2
```

## commit提交容器成镜像

```
docker commit -c 'CMD ["/bin/sh","-c","/bin/httpd -f -h /data/web/html"]'  "ID"
```

## 提交镜像给hub.docker 

```
docker login
docker push shaoxh/myimages:v20
```

## 查看网桥的信息

```
 docker network inspect bridge
```

## 根据一个容器网路启动

```
docker run -it --restart=always --network container:acff ubuntu:focal
```

## 共享宿主机的网络

```
docker run -it --restart=always --network host ubuntu:focal
```

## 定义容器的主机名

```
 docker run --name bbox2 -it --rm --hostname shaoxh busybox
```

## 定义解析主机名

```
 docker run --name bbox2 -it --rm --hostname shaoxh --add-host gw.shaoxh.com:172.17.0.1 busybox   add-host可重复使用

/ # cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.1      gw.shaoxh.com
172.17.0.4      shaoxh
/ #

```

## 添加dns和搜索域

```
docker run --name bbox2 -it --rm --hostname shaoxh --add-host gw.shaoxh.com:172.17.0.1 --dns 172.17.0.1 --dns-search ilinux.io busybox

/ # cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.1      gw.shaoxh.com
172.17.0.4      shaoxh
/ # cat /etc/resolv.conf
search ilinux.io
nameserver 172.17.0.1
```

## 自己定义一个网桥

### 查看网络模式

```
docker network ls
```

### 创建一个新的bridge网络

```
docker network create --driver bridge --subnet=172.18.18.0/16 --gateway=172.18.18.1 mynet
```

### 查看网络信息

```
docker network inspect mynet
```

### 创建容器并指定容器ip

```
 docker run -e TZ="Asia/Shanghai" --privileged -itd -h test03.com --name test03 --network=mynet --ip 172.18.18.3 centos /usr/sbin/init

```

### 运行容器

```
 docker exec -it test03 /bin/bash
```

### centos最小化安装没有ifconfig命令，可通过yum进行安装

```
yum install -y net-tools
```

### 安装ssh服务

```
yum install -y openssh-server

yum install -y openssh-clients

systemctl start sshd.service
```

### 新增非root用户

```
useradd shaoxh

passwd shaoxh
```

## 端口映射

### 随机端口映射

```
docker run -d -P --name web1 nginx
```

### 指定端口映射

```
docker run -d -p 80:80 --name web2 nginx
```

### 指定地址的端口映射

```
docker run -d --name web1 -p 192.168.0.111:80:80 nginx
```

### 指定地址，不固定端口的映射

```
 docker run -d --name web1 -p 192.168.0.111::80 nginx
```

### 查看端口映射

```
[root@mon01 ~]# docker container port web1
80/tcp -> 192.168.0.111:49153
```

## 创建网桥

```
docker network create --subnet 10.0.0.0/24 mybr0
docker run -d --name web1 --network mybr0 -P nginx

```

### 容器连接别的网桥

```
 docker run -it --network mybr0 --name web1  centos
 docker network connect bridge web1
```

## 修改默认的docker0桥的ip地址

```
[root@mon01 ~]# cat /etc/docker/daemon.json 
{
"registry-mirrors": ["https://5u3371wf.mirror.aliyuncs.com"],
"bip": "172.31.0.1/16"
}
```

## 容器卷

```
docker run -itd --name dbdata -v /data centos
docker inspect dbdata
进入容器拷贝文件到 /data/ 目录下
docker exec -it dbdata /bin/bash
cp /etc/passwd /data

宿主机查看本地容器卷的内容
[root@mon01 _data]# pwd
/var/lib/docker/volumes/a23227bca88a47da6df3b7b3fcd3246cb420be5fccf745b6184ca060ae9ec509/_data
[root@mon01 _data]# ls
passwd

```

## Dockerfile

```
[root@mon01 ~]# mkdir build_workshop
[root@mon01 ~]# cd build_workshop
[root@mon01 build_workshop]# vim Dockerfile
[root@mon01 build_workshop]# 
[root@mon01 build_workshop]# wget www.baidu.com
--2021-05-03 20:32:51--  http://www.baidu.com/
正在解析主机 www.baidu.com (www.baidu.com)... 112.80.248.75, 112.80.248.76
正在连接 www.baidu.com (www.baidu.com)|112.80.248.75|:80... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：2381 (2.3K) [text/html]
正在保存至: “index.html”

100%[================================================================================================================================>] 2,381       --.-K/s 用时 0s      

2021-05-03 20:32:52 (413 MB/s) - 已保存 “index.html” [2381/2381])

[root@mon01 build_workshop]# ls
Dockerfile  index.html
[root@mon01 build_workshop]# 
[root@mon01 build_workshop]# cat Dockerfile 
FROM busybox:latest
LABEL maintainer="magedu <mage@magedu.com>"
COPY index.html /data/web/html/

[root@mon01 build_workshop]# docker build . -t myimg:v0.1

```

### 使用通配符

```
[root@mon01 build_workshop]# ls
Dockerfile  pages
[root@mon01 build_workshop]# cat Dockerfile 
FROM busybox:latest
LABEL maintainer="magedu <mage@magedu.com>"
COPY pages/*.html /data/web/html/
[root@mon01 build_workshop]# cat .dockerignore 
pages/test2.html

```

### add命令

```
[root@mon01 build_workshop]# wget http://nginx.org/download/nginx-1.14.2.tar.gz

查看dockerfile的内容
[root@mon01 build_workshop]# cat Dockerfile 
FROM busybox:latest
LABEL maintainer="magedu <mage@magedu.com>"
COPY pages/*.html /data/web/html/
ADD http://nginx.org/download/nginx-1.15.8.tar.gz /tmp/
ADD nginx-1.14.2.tar.gz /usr/src

如果add 指定的是url 会从网络上下载压缩包，自动下载目标路径中
ADD http://nginx.org/download/nginx-1.15.8.tar.gz /tmp/

如果add是本地文件，会自动解压到目标路径中
ADD nginx-1.14.2.tar.gz /usr/src

如下所示：
/ # ls /tmp/
nginx-1.15.8.tar.gz
/ # ls /usr/src
nginx-1.14.2
/ # ls /tmp
nginx-1.15.8.tar.gz

```

### WORKDIR

```
用于为Dockfile中所有的RUN、CMD、ENTRYPOINT、COPY、和ADD指定设定工作目录
[root@mon01 build_workshop]# cat Dockerfile 
FROM busybox:latest
LABEL maintainer="magedu <mage@magedu.com>"
COPY pages/*.html /data/web/html/
ADD http://nginx.org/download/nginx-1.15.8.tar.gz /tmp/
WORKDIR /usr/
ADD nginx-1.14.2.tar.gz src/

```

### VOLUME

```
volume 用于在image中创建一个挂载点目录，以挂载Dockerfile host上的卷或其他容器上的卷
如果挂载点目录路径下此前在文件存在，docker run 命令会在卷挂载完成后将此前的所有文件复制到新挂载的卷中。
[root@mon01 build_workshop]# cat Dockerfile 
FROM busybox:latest
LABEL maintainer="magedu <mage@magedu.com>"
COPY pages/*.html /data/web/html/
ADD http://nginx.org/download/nginx-1.15.8.tar.gz /tmp/
WORKDIR /usr/
ADD nginx-1.14.2.tar.gz src/
VOLUME /data/web/html/


```

### EXPOSE

```
用于为容器打开指定要监听的端口以实现与外部通信
[root@mon01 build_workshop]# cat Dockerfile 
FROM busybox:latest
LABEL maintainer="magedu <mage@magedu.com>"
COPY pages/*.html /data/web/html/
ADD http://nginx.org/download/nginx-1.15.8.tar.gz /tmp/
WORKDIR /usr/
ADD nginx-1.14.2.tar.gz src/
VOLUME /data/web/html/
EXPOSE 80/tcp

[root@mon01 build_workshop]# docker build . -t myimg:v0.6
[root@mon01 build_workshop]# docker run -it myimg:v0.6
/usr # httpd -h /data/web/html/
/usr # netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 :::80                   :::*                    LISTEN      7/httpd
/usr # 

使用elinks  访问 


```

### ENV

```
用于为镜像定义所需要的环境变量，并可被Dockerfile文件中位于其后的其他指令（如ENV、ADD、COPY等）所调用
[root@mon01 build_workshop]# cat Dockerfile 
FROM busybox:latest
ENV webhome="/data/web/html"
LABEL maintainer="magedu <mage@magedu.com>"
COPY pages ${webhome}
ADD http://nginx.org/download/nginx-1.15.8.tar.gz ${webhome}
WORKDIR ${webhome}
ADD nginx-1.14.2.tar.gz ./
VOLUME ${webhome}
EXPOSE 80/tcp
[root@mon01 build_workshop]# docker build . -t myimg:v0.8

```

### ARG

```
arg 可以build 时 直接向变量传值
[root@mon01 build_workshop]# cat Dockerfile 
FROM busybox:latest
ARG webhome="/data/web/html"
LABEL maintainer="magedu <mage@magedu.com>"
COPY pages ${webhome}
ADD http://nginx.org/download/nginx-1.15.8.tar.gz ${webhome}
WORKDIR ${webhome}
ADD nginx-1.14.2.tar.gz ./
VOLUME ${webhome}
EXPOSE 80/tcp
[root@mon01 build_workshop]# docker build --build-arg webhome="/web/htdocs/" . -t myimge:v0.9


```

### RUN

```
用于指定docker build 过程中运行的程序，其可以是任何命令
[root@mon01 build_workshop]# cat Dockerfile
FROM busybox:latest
ARG webhome="/data/web/html"
LABEL maintainer="magedu <mage@magedu.com>"
COPY pages ${webhome}
ADD http://nginx.org/download/nginx-1.15.8.tar.gz ${webhome}
WORKDIR ${webhome}
ADD nginx-1.14.2.tar.gz ./
VOLUME ${webhome}
EXPOSE 80/tcp
RUN mkdir -p /web/bbs/ && \
  echo hi > /web/bbs/index.html

[root@mon01 build_workshop]# docker build . -t myimg:v1.0

```

```
[root@mon01 ap]# pwd
/root/ap
[root@mon01 ap]# cat Dockerfile
FROM centos:7
ARG docroot=/var/www/html/
RUN yum makecache && \
        yum -y install httpd php php-mysql && \
        yum clean all && \
        rm -rf /var/cache/yum/*
[root@mon01 ap]# docker run --name web1 --rm -it php-httpd:v0.1 /bin/bash
```

### CMD

```
类似于RUN指令，CMD指令也可用于运行任何命令或应用程序，不过，二者的运行时间点不同
[root@mon01 ap]# cat Dockerfile
FROM centos:7
ARG docroot=/var/www/html/
RUN yum makecache && \
        yum -y install httpd php php-mysql && \
        yum clean all && \
        rm -rf /var/cache/yum/*
CMD ["/usr/sbin/httpd","-DFOREGROUND"]

```

### ENTRYPOINT 

```
[root@mon01 ap]# pwd
/root/ap

[root@mon01 ap]# cat Dockerfile
FROM centos:7
ARG docroot=/var/www/html/
RUN yum makecache && \
        yum -y install httpd php php-mysql && \
        yum clean all && \
        rm -rf /var/cache/yum/*
ADD phpinfo.php ${docroot}
ADD entrypoint.sh /bin/
EXPOSE 80/tcp
VOLUME ${docroot}
CMD ["/usr/sbin/httpd","-DFOREGROUND"]
ENTRYPOINT ["/bin/entrypoint.sh"]

有执行权限的脚本
[root@mon01 ap]# cat entrypoint.sh
#!/bin/bash
#
#listen_port=${LISTEN_PORT:-80}
server_name=${SERVER_NAME:-localhost}
doc_root=${DOC_ROOT:-/var/www/html/}
cat > /etc/httpd/conf.d/myweb.conf <<EOF
#Listen $listen_port
<VirtualHost *:80>
        ServerName $server_name
        DocumentRoot $doc_root
        <Directory $doc_root>
                Options none
                AllowOverride none
                Require all granted
        </Directory>
</Virtualhost>
EOF

exec "$@"

php测试页
[root@mon01 ap]# cat phpinfo.php
<?php
        phpinfo();
?>

[root@mon01 ap]# docker  run -it php-httpd:v0.6
[root@mon01 ap]# docker  run  --name myweb -P -it php-httpd:v0.6

```

### USER

```
用于指定运行image时的或运行Dockerfile 中 任何RUN，CMD或ENTRYPOINT指令指定的程序时的用户名或UID
```

### HEALTHCHECK

```
[root@mon01 ap]# cat Dockerfile 
FROM centos:7
ARG docroot=/var/www/html/
RUN yum makecache && \
        yum -y install httpd php php-mysql && \
        yum clean all && \
        rm -rf /var/cache/yum/*
ADD ok.html phpinfo.php ${docroot}
ADD entrypoint.sh /bin/
EXPOSE 80/tcp
VOLUME ${docroot}
HEALTHCHECK --interval=3s --timeout=3s --start-period=2s CMD curl -f http://localhost/ok.html || exit 1
CMD ["/usr/sbin/httpd","-DFOREGROUND"]
ENTRYPOINT ["/bin/entrypoint.sh"]

[root@mon01 ap]# cat ok.html 
ok

[root@mon01 ~]# docker ps -l
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS                     PORTS     NAMES
492762cd3e63   php-httpd:v0.7   "/bin/entrypoint.sh …"   4 minutes ago   Up 4 minutes (unhealthy)   80/tcp    vibrant_sanderson

```

### SHELL

```

```

### STOPSIGNAL

```

```

### ONBUILD

```
[root@mon01 ~]# mkdir testimg
[root@mon01 ~]# cd testimg/
[root@mon01 testimg]# vim Dockerfile
[root@mon01 testimg]# cat Dockerfile 
FROM busybox:latest
LABEL mantainer="magedu <mage@magedu.com>"
RUN mkdir -p /data/web/
ONBUILD ADD http://nginx.org/download/nginx-1.15.8.tar.gz /usr/src/
[root@mon01 testimg]# docker build . -t testing:v0.1

[root@mon01 ~]# mkdir secimg
[root@mon01 ~]# cd secimg
[root@mon01 secimg]# vim Dockerfile
[root@mon01 secimg]# 
[root@mon01 secimg]# cat Dockerfile 
FROM testing:v0.1
LABEL hi=hello

[root@mon01 secimg]# docker build . -t secimg:v0.1

自己做镜像时不会执行，别人在基于你的镜像时会执行 （命令的延迟运行）
```

### SAVE 镜像打包保存

```
[root@mon01 secimg]# docker  save myimg:v0.6 php-httpd:v0.6 -o /root/myimages.tar

```

### LOAD 镜像加载

```
[root@mon02 ~]# docker load -i myimages.tar 
```

## Docker Registry 

```
Registry用于保存docker镜像，包括镜像的层次结构和元数据
用户可自建registry，也可用使用官方的Docker Hub
分类
	Sponsor Registry： 第三方的registry，供客户和Docker社区使用
	Mirror Registry： 第三方的registry， 只让客户使用
	Vendor Registry：由发布Docker 镜像的供应商提供的 registry
	Private Registry: 通过设有防火墙和恩外的安全层的私有实体的registry
```

### 搭建私有仓库

```
[root@mon02 ~]# yum -y install docker-distribution
[root@mon02 ~]# rpm -ql docker-distribution
/etc/docker-distribution/registry/config.yml
/usr/bin/registry
/usr/lib/systemd/system/docker-distribution.service
/usr/share/doc/docker-distribution-2.6.2
/usr/share/doc/docker-distribution-2.6.2/AUTHORS
/usr/share/doc/docker-distribution-2.6.2/CONTRIBUTING.md
/usr/share/doc/docker-distribution-2.6.2/LICENSE
/usr/share/doc/docker-distribution-2.6.2/MAINTAINERS
/usr/share/doc/docker-distribution-2.6.2/README.md
/var/lib/registry

查看配置文件
[root@mon02 ~]# cat /etc/docker-distribution/registry/config.yml
version: 0.1
log:
  fields:
    service: registry
storage:
    cache:
        layerinfo: inmemory
    filesystem:
        rootdirectory: /var/lib/registry
http:
    addr: :5000

启动服务
[root@mon02 ~]# systemctl start docker-distribution

查看端口5000
[root@mon02 ~]# ss -tnl

上传一个镜像首先修改标签：
[root@mon01 ~]# docker tag myimg:v0.4 192.168.0.112:5000/myimg:v0.4
、
默认使用的是https协议
[root@mon01 ~]# docker push 192.168.0.112:5000/myimg:v0.4
The push refers to repository [192.168.0.112:5000/myimg]
Get https://192.168.0.112:5000/v2/: dial tcp 192.168.0.112:5000: connect: no route to host

修改docker参数，使用不安全的镜像仓库
[root@mon01 ~]# cat /etc/docker/daemon.json
{
"registry-mirrors": ["https://5u3371wf.mirror.aliyuncs.com"],
"bip": "172.31.0.1/16",
"insecure-registries": ["192.168.0.112:5000"]
}

重启服务
[root@mon01 ~]# systemctl restart docker

再次上传
[root@mon01 ~]# docker push 192.168.0.112:5000/myimg:v0.4

The push refers to repository [192.168.0.112:5000/myimg]
f31683a27e52: Pushed
2672a39e6466: Pushed
20508e9281cd: Pushed
36b45d63da70: Pushed
v0.4: digest: sha256:203291eef9f3b3f53654374ec2a51d068fafa16e2aa4b2f24a6af8e786a576a5 size: 1158

安装docker
[root@mon01 yum.repos.d]# scp docker-ce.repo 192.168.0.112:/etc/yum.repos.d/
[root@mon02 registry]# yum -y install docker-ce
[root@mon01 ~]# scp /etc/docker/daemon.json 192.168.0.112:/etc/docker/
[root@mon02 docker]# systemctl restart docker
[root@mon02 docker]# docker pull 192.168.0.112:5000/myimg:v0.4
[root@mon02 docker]# docker images
REPOSITORY                 TAG       IMAGE ID       CREATED        SIZE
192.168.0.112:5000/myimg   v0.4      174c5dd57b65   15 hours ago   8.46MB

第三方镜像下载
[root@mon02 docker]# docker pull quay.io/coreos/flannel:v0.13.0-amd64
```



### harbor

```
[root@mon02 docker]# systemctl stop docker-distribution

[root@mon02 docker]# yum -y install docker-compose

下载harbor 
[root@mon02 ~]# tar xf harbor-offline-installer-v1.7.5.tgz -C /usr/local/
[root@mon02 ~]# cd /usr/local/harbor/

编辑配置文件cfg
[root@mon02 harbor]# vim harbor.cfg 
hostname = 192.168.0.112

安装
[root@mon02 harbor]# ./install.sh 
......
✔ ----Harbor has been installed and started successfully.----

会监听服务器的80和443，4443端口
[root@mon02 harbor]# ss -tnl

浏览器访问
http://192.168.0.112
账号： admin
密码：Harbor12345

Harbor用户管理
      创建用户
       用户名：        shaoxh
       邮箱：         shaoxh@magedu.com
       全名：         shaoxianheng
       密码：         Xianheng@123
       确认密码       Xianheng@123
       注释：         shaoxh.com
       
然后退出使用普通用户shaoxh登录
新建项目
      prod 公开
      
推送镜像到harbor
    修改daemon.jason 配置文件的仓库接口
[root@mon01 ~]# cat /etc/docker/daemon.json 
{
"registry-mirrors": ["https://5u3371wf.mirror.aliyuncs.com"],
"bip": "172.31.0.1/16",
"insecure-registries": ["192.168.0.112"]
}

重启docker
[root@mon01 ~]# systemctl restart docker



```

### 登录harbor

```

[root@mon01 harbor]# docker login 192.168.0.112
Username: shaoxh
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded


docker push 192.168.0.112/prod/myalpine:v1

```

## lorel/docker-stress-ng 压测

```
cpu压测
[root@node1 ~]# docker run --name stress --cpus 2 --rm lorel/docker-stress-ng -c 2
 
 --cpuset-cpus 0,2 指定使用那个cpu
 --cpu--share 1024
[root@node1 ~]# docker stats

内存压测
[root@node1 ~]# docker run --name stress -m 512m --rm lorel/docker-stress-ng -vm 2

```

