# Kubernetes安装部署文档

## 1.1.1 环境准备

```
    （1）借助于NTP服务设定各节点时间精确同步；
    （2）通过DNS完成各节点的主机名称解析，测试环境主机数量较少时也可以使用hosts文件进行；
    （3）关闭各节点的iptables或firewalld服务，并确保它们被禁止随系统引导过程启动；
    （4）各节点禁用SELinux； 
    （5）各节点禁用所有的Swap设备；
    （6）若要使用ipvs模型的proxy，各节点还需要载入ipvs相关的各模块；
```

### 1.1.2 配置hosts文件

```
cat >> /etc/hosts <<EOF
192.168.0.111 master.magedu.com master
192.168.0.112 node2.magedu.com node2
192.168.0.113 node3.magedu.com node3
192.168.0.114 node4.magedu.com node4
EOF
```

### 1.1.3 关闭防火墙

```
systemctl disable firewalld
systemctl stop firewalld
```

### 1.1.4 关闭selinux

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config 
setenforce 0

```

### 1.1.5 关闭Swap设备

```
swapoff -a
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab

```

### 1.1.6  要载入ipvs相关的各模块

```
ls /usr/lib/modules/3.10.0-693.el7.x86_64/kernel/net/netfilter/ipvs

cat > /etc/sysconfig/modules/ipvs.modules << EOF
#!/bin/bash
ipvs_mods_dir="/usr/lib/modules/$(uname -r)/kernel/net/netfilter/ipvs"
for mod in $(ls $ipvs_mods_dir | grep -o "^[^.]*"); do
    /sbin/modinfo -F filename $mod  &> /dev/null
    if [ $? -eq 0 ]; then
        /sbin/modprobe $mod
    fi
done
EOF
chmod +x /etc/sysconfig/modules/ipvs.modules

```

### 1.1.7 安装docker-ce

```
# 使用阿里云镜像仓库 
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
```

```
yum -y install docker-ce

```

### 1.1.8 iptables的FORWARD默认策略为DROP

```
sed  -i '/ExecStart/aExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT' /usr/lib/systemd/system/docker.service

下面使用代理的方法，
sed -i '/ExecStart=/iEnvironment="HTTPS_PROXY=http://www.ik8s.io:10070"' /usr/lib/systemd/system/docker.service
sed -i '/ExecStart=/iEnvironment="NO_PROXY=127.0.0.1/8,192.168.0.5/24"' /usr/lib/systemd/system/docker.service

```

### 1.1.9启动docker

```
 systemctl daemon-reload 
 systemctl start docker && systemctl enable docker 
 docker info
 
```

### 1.2.1查看一下iptables

```
iptables -vnL

```

```
sysctl -a |grep bridge
cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl -p /etc/sysctl.d/k8s.conf



```

### 1.2.2 安装 kubeadm, kubelet 和 kubectl

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum repolist
yum list all |grep "^kube"
yum -y install kubeadm kubectl kubelet

```

```
rpm -ql kubelet
rpm -ql kubeadm

```

```
[root@node1 ~]# cat /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--fail-swap-on=false"

```

```
 kubeadm config -h
 kubeadm config print -h
 kubeadm config print init-defaults

```

## 2.1.1 (master 节点) docker下载镜像

```
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.19.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.19.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.19.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.19.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.13-0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.7.0
```

### 2.1.2 根据配置文件初始化集群(在master上操作)

```
# 生成kubeadm配置文件
cat > kubeadm-master.config <<EOF
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kubernetesVersion: v1.19.3
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
EOF


#初始化k8s集群
kubeadm init --config=kubeadm-master.config
```

```
mkdir .kube
cp /etc/kubernetes/admin.conf .kube/config
 kubectl get nodes
 
 
 下载flannel.yml 文件去github.com/coros/
 kubectl apply -f kube-flannel.yml 
 kubectl get pods -n kube-system
 kubectl get nodes

```



## 3.1.1 node节点操作

在执行一次1章节的安装过程，然后加入集群；

```
[root@node4 ~]# kubeadm join 192.168.0.111:6443 --token 18xp1l.uxuo0p3uiif5jfi1  --discovery-token-ca-cert-hash sha256:159a0755b2a58cf9d97c40a6c68f849b099d24f7dc64eb49665bc
ff1ddb944b7  --ignore-preflight-errors=Swap
```

## 4.1.1 创建pod

```
[root@node1 ~]# kubectl get nodes
[root@node1 ~]# kubectl get ns
[root@node1 ~]# kubectl get pods -n kube-system
root@node1 ~]# kubectl get pods -n kube-system -o wide

```

### 4.1.2 查看主机上又那些种类的资源

```
[root@node1 ~]# kubectl api-resources

```

### 4.13 查看deployed中的控制器

```
[root@node1 ~]# kubectl get deploy -n kube-system
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           17h

```

### 4.1.4 创建develop 的资源

```
[root@node1 ~]# kubectl create namespace develop
[root@node1 ~]# kubectl create namespace testing
[root@node1 ~]# kubectl create namespace prod

```

### 4.1.5 查看名称空间

```
[root@node1 ~]# kubectl get ns

```

### 4.1.6 删除一个名称空间

```
[root@node1 ~]# kubectl delete namespace develop
[root@node1 ~]# kubectl get ns
[root@node1 ~]# kubectl delete  ns/testing ns/prod

```

### 4.1.7 查看名称空间

```
root@node1 ~]# kubectl get ns/default
[root@node1 ~]# kubectl get ns/default -o wide
[root@node1 ~]# kubectl get ns/default -o yaml
[root@node1 ~]# kubectl get ns/default -o json


```

### 4.1.8 描述信息，当前的状态信息

```
[root@node1 ~]# kubectl describe ns/default

```

### 4.1.9 创建一个nginx的pod

```
[root@node1 ~]# kubectl create deploy ngx-dep --image=nginx:1.14-alpine

```

### 4.2.1 显示所有的资源

```
[root@node1 ~]# kubectl get all

```

### 4.2.2 查看pods

```
[root@node1 ~]# kubectl get pods
[root@node1 ~]# kubectl get pods -o json|grep IP
                                "f:hostIP": {},
                                "f:podIP": {},
                                "f:podIPs": {
                "hostIP": "192.168.0.114",
                "podIP": "10.244.3.2",
                "podIPs": [
[root@node1 ~]# curl 10.244.3.2


```

### 4.2.3 删除pod

```
[root@node1 ~]# kubectl delete pods/ngx-dep-5c8d96d457-5d4dm
[root@node1 ~]# kubectl get pods -o json|grep IP
                                "f:hostIP": {},
                                "f:podIP": {},
                                "f:podIPs": {
                "hostIP": "192.168.0.112",
                "podIP": "10.244.1.2",
                "podIPs": [
[root@node1 ~]# curl 10.244.1.2

```

### 4.24 创建service

```
[root@node1 ~]# kubectl create service clusterip nginx-svc --tcp=80:80
[root@node1 ~]# kubectl get svc
[root@node1 ~]# kubectl delete svc/nginx-svc

[root@node1 ~]#  kubectl create namespace develop

[root@node1 ~]# kubectl create service clusterip ngx-dep --tcp=80:80
[root@node1 ~]# kubectl get svc/ngx-dep -o yaml
[root@node1 ~]# kubectl describe svc/ngx-dep
Name:              ngx-dep
Namespace:         default
Labels:            app=ngx-dep
Annotations:       <none>
Selector:          app=ngx-dep
Type:              ClusterIP
IP:                10.110.114.226
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.2:80
Session Affinity:  None
Events:            <none>

[root@node1 ~]# curl 10.110.114.226





```

### 4.2.5 删除pod

```
[root@node1 ~]# kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
ngx-dep-5c8d96d457-5bpz9   1/1     Running   0          25m
[root@node1 ~]# kubectl delete pods ngx-dep-5c8d96d457-5bpz9 
pod "ngx-dep-5c8d96d457-5bpz9" deleted
[root@node1 ~]# kubectl describe svc/ngx-dep
Name:              ngx-dep
Namespace:         default
Labels:            app=ngx-dep
Annotations:       <none>
Selector:          app=ngx-dep
Type:              ClusterIP
IP:                10.110.114.226
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.2.2:80
Session Affinity:  None
Events:            <none>
[root@node1 ~]# curl 10.110.114.226


```

### 4.2.6 查看dns

```
[root@node1 ~]# kubectl get svc -n kube-system
    10.96.0.10 
    
[root@node1 ~]# cat /etc/resolv.conf 
# Generated by NetworkManager
nameserver 10.96.0.10

[root@node1 ~]# curl ngx-dep.default.svc.cluster.local.

```

### 4.2.7  查看svc

```
[root@node1 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   18h
ngx-dep      ClusterIP   10.110.114.226   <none>        80/TCP    20m
[root@node1 ~]# kubectl delete svc/ngx-dep
service "ngx-dep" deleted
[root@node1 ~]# kubectl create service clusterip ngx-dep --tcp=80:80
service/ngx-dep created
[root@node1 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   18h
ngx-dep      ClusterIP   10.109.12.122   <none>        80/TCP    4s
[root@node1 ~]# 
[root@node1 ~]# curl ngx-dep.default.svc.cluster.local.

```

### 4.2.8 下载myapp

```
[root@node1 ~]# kubectl create deploy myapp --image=ikubernetes/myapp:v1
deployment.apps/myapp created
[root@node1 ~]# kubectl get deploy
[root@node1 ~]# kubectl get pods
[root@node1 ~]# kubectl get pods -o wide
[root@node1 ~]# curl 10.244.1.3
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>
[root@node1 ~]# curl 10.244.1.3/hostname.html
myapp-7d4b7b84b-stfvc
[root@node1 ~]# 
[root@node1 ~]# kubectl create service clusterip myapp --tcp=80:80
[root@node1 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   18h
myapp        ClusterIP   10.111.172.97   <none>        80/TCP    29s
ngx-dep      ClusterIP   10.109.12.122   <none>        80/TCP    25m
[root@node1 ~]# 
[root@node1 ~]# curl myapp.default.svc.cluster.local.
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>
[root@node1 ~]# 

[root@node1 ~]# curl myapp.default.svc.cluster.local/hostname.html




```

### 4.2.9 实现扩容

```
[root@node1 ~]# kubectl scale --replicas=3 deployment myapp
[root@node1 ~]# kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
myapp-7d4b7b84b-fwcgz      1/1     Running   0          64s
myapp-7d4b7b84b-s95x7      1/1     Running   0          64s
myapp-7d4b7b84b-stfvc      1/1     Running   0          26m
ngx-dep-5c8d96d457-lq785   1/1     Running   0          48m

[root@node1 ~]# kubectl get pods -o wide
[root@node1 ~]# kubectl describe svc/myapp

随机调度
[root@node1 ~]# curl myapp.default.svc.cluster.local/hostname.html
myapp-7d4b7b84b-s95x7
[root@node1 ~]# curl myapp.default.svc.cluster.local/hostname.html
myapp-7d4b7b84b-fwcgz
[root@node1 ~]# curl myapp.default.svc.cluster.local/hostname.html
myapp-7d4b7b84b-stfvc
[root@node1 ~]# curl myapp.default.svc.cluster.local/hostname.html
myapp-7d4b7b84b-s95x7
[root@node1 ~]# curl myapp.default.svc.cluster.local/hostname.html
myapp-7d4b7b84b-fwcgz
[root@node1 ~]# curl myapp.default.svc.cluster.local/hostname.html
myapp-7d4b7b84b-stfvc


```

### 4.3.1 实现缩容

```
[root@node1 ~]# kubectl scale --replicas=2 deployment myapp
[root@node1 ~]# kubectl get pods
[root@node1 ~]# kubectl describe svc/myapp

```

### 4.3.2 实现映射端口访问

```
root@node1 ~]# kubectl delete svc/myapp
[root@node1 ~]# kubectl create service nodeport myapp --tcp=80:80
service/myapp created
[root@node1 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        19h
myapp        NodePort    10.105.191.140   <none>        80:30821/TCP   6s
ngx-dep      ClusterIP   10.109.12.122    <none>        80/TCP         38m
[root@node1 ~]# 

浏览器访问 192.168.0.111:30821  在每台主机上都可以访问
```

### 4.3.3 显示群组

```
[root@node1 ~]# kubectl api-versions

```

### 4.4.1 定义一个yaml文件

```
[root@node1 basic]# kubectl get ns default -o yaml 
[root@node1 basic]# kubectl get ns
[root@node1 basic]# cat develop-ns.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: testing
  
[root@node1 basic]# kubectl create -f develop-ns.yaml       陈述式对象配置 
[root@node1 basic]# cp develop-ns.yaml prod-ns.yaml
[root@node1 basic]# vim prod-ns.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: prod
[root@node1 basic]# kubectl apply -f prod-ns.yaml          声明式对象配置 可以重复执行


```



### 4.4.2 创建一个自主可控的pod

```
[root@node1 basic]# kubectl get pods myapp-7d4b7b84b-fwcgz -o yaml > pod-demo.yaml

修改模板文件里内容
[root@node1 basic]# cat pod-demo.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: pod-demo
  namespace: develop
spec:
  containers:
  - image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    name: myapp
    resources: {}
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  
 [root@node1 basic]# kubectl apply -f pod-demo.yaml
  pod/pod-demo created   没有语法错误表示成功
  
  查看创建好的pod
  [root@node1 basic]# kubectl get pods -n develop


```

### 4.4.3 帮助文档

```
[root@node1 basic]# kubectl explain pods

  metadata	<Object>   这样标记的可以加点查看
[root@node1 basic]# kubectl explain pods.metadata



```

### 4.4.4 一个pod两个镜像

```
[root@node1 basic]# cat pod-demo-2.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: prod
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
  - name: bbox
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c","sleep 86400"]


查看状态
[root@node1 basic]# kubectl get pods -n prod
NAME       READY   STATUS    RESTARTS   AGE
pod-demo   2/2     Running   0          3m39s

[root@node1 basic]# kubectl get pods -n prod -o yaml

```

### 4.5.6 进入box

```
[root@node1 basic]# kubectl exec pod-demo -c bbox -n prod -it -- /bin/sh
/ # wget -O - -q 127.0.0.1
[root@node1 ~]# kubectl logs  pod-demo -n prod myapp

```

### 4.5.7 查看日志

```
[root@node1 ~]# kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
myapp-7d4b7b84b-fwcgz      1/1     Running   0          4h26m
myapp-7d4b7b84b-stfvc      1/1     Running   0          4h52m
ngx-dep-5c8d96d457-lq785   1/1     Running   0          5h13m
prod                       1/1     Running   0          53m
[root@node1 ~]# kubectl logs myapp-7d4b7b84b-fwcgz

```

### 4.5.8 共享宿主机的网络空间

```
[root@node1 basic]#  kubectl explain pods.spec

[root@node1 basic]# cat host-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
  hostNetwork: true
  
[root@node1 basic]# kubectl apply -f host-pod.yaml 
[root@node1 basic]# kubectl get pods -o wide

用浏览器访问：

```

```
[root@node1 basic]# kubectl explain pods.spec.containers.ports


```

### 4.5.9 删除pod

```
[root@node1 basic]# kubectl delete -f host-pod.yaml 
pod "mypod" deleted

```

### 4.6.1 映射pod到宿主机的端口

```
[root@node1 basic]# cat host-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports: 
    - protocol: TCP
      containerPort: 80
      name: http
      hostPort: 8080

[root@node1 basic]# kubectl get pods -o wide|awk '{print $1,$6,$7}'
NAME IP NODE
myapp-7d4b7b84b-fwcgz 10.244.2.3 node3
myapp-7d4b7b84b-stfvc 10.244.1.3 node2
mypod 10.244.2.4 node3
ngx-dep-5c8d96d457-lq785 10.244.2.2 node3
prod 10.244.1.4 node2

浏览器访问 http://192.168.0.113:8080

```

### 4.6.2 查看pod的默认标签

```
[root@node1 ~]# kubectl get pods --show-labels
[root@node1 ~]# kubectl get pods -n prod --show-labels


```

### 4.6.3 添加标签

```
[root@node1 ~]# cat pod-demo-2.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: prod
  labels:
    app: pod-demo
    rel: stable
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
  - name: bbox
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c","sleep 86400"]
    
    
[root@node1 ~]# kubectl apply -f pod-demo-2.yaml
[root@node1 ~]# kubectl get pods -n prod --show-labels

```

### 4.6.4 label命令

```
[root@node1 ~]# kubectl label pods pod-demo -n prod tier=frontend
pod/pod-demo labeled
[root@node1 ~]# kubectl get pods -n prod --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
pod-demo   2/2     Running   0          4h23m   app=pod-demo,rel=stable,tier=frontend
[root@node1 ~]# 

```



### 4.6.5 覆盖app的标签 --overwrite

```
[root@node1 ~]# kubectl label pods pod-demo -n prod --overwrite app=myapp

```

### 4.6.6  删除一个标签

```
[root@node1 ~]# kubectl label pods pod-demo -n prod rel-
pod/pod-demo labeled
[root@node1 ~]# kubectl get pods -n prod --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
pod-demo   2/2     Running   0          4h32m   app=myapp,tier=frontend

```

### 4.6.7 -l 使用选择器条件

```
[root@node1 ~]# kubectl get pods --show-labels
NAME                       READY   STATUS    RESTARTS   AGE     LABELS
myapp-7d4b7b84b-fqqjt      1/1     Running   0          5h25m   app=myapp,pod-template-hash=7d4b7b84b
myapp-7d4b7b84b-prl6g      1/1     Running   0          5h39m   app=myapp,pod-template-hash=7d4b7b84b
ngx-dep-5c8d96d457-lgr52   1/1     Running   0          5h47m   app=ngx-dep,pod-template-hash=5c8d96d457
[root@node1 ~]# kubectl get pods --show-labels -l app=myapp
NAME                    READY   STATUS    RESTARTS   AGE     LABELS
myapp-7d4b7b84b-fqqjt   1/1     Running   0          5h25m   app=myapp,pod-template-hash=7d4b7b84b
myapp-7d4b7b84b-prl6g   1/1     Running   0          5h39m   app=myapp,pod-template-hash=7d4b7b84b
[root@node1 ~]# kubectl get pods --show-labels -l app!=myapp
NAME                       READY   STATUS    RESTARTS   AGE     LABELS
ngx-dep-5c8d96d457-lgr52   1/1     Running   0          5h48m   app=ngx-dep,pod-template-hash=5c8d96d457

```

### 4.6.8 使用或者条件

```
[root@node1 ~]# kubectl get pods --show-labels -l "app in (myapp,ngx-dep)" 
NAME                       READY   STATUS    RESTARTS   AGE     LABELS
myapp-7d4b7b84b-fqqjt      1/1     Running   0          5h29m   app=myapp,pod-template-hash=7d4b7b84b
myapp-7d4b7b84b-prl6g      1/1     Running   0          5h44m   app=myapp,pod-template-hash=7d4b7b84b
ngx-dep-5c8d96d457-lgr52   1/1     Running   0          5h52m   app=ngx-dep,pod-template-hash=5c8d96d457
[root@node1 ~]# kubectl get pods -L app -l "app in (myapp,ngx-dep)" 
NAME                       READY   STATUS    RESTARTS   AGE     APP
myapp-7d4b7b84b-fqqjt      1/1     Running   0          5h30m   myapp
myapp-7d4b7b84b-prl6g      1/1     Running   0          5h44m   myapp
ngx-dep-5c8d96d457-lgr52   1/1     Running   0          5h52m   ngx-dep

```

```
[root@node1 ~]# kubectl get pods  --show-labels -l app 
[root@node1 ~]# kubectl get pods  --show-labels -l '!app' 
No resources found in default namespace.

```



