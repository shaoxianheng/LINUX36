## [GitHub]()

## 1.1 git 安装

```
[root@nfs ~]# yum -y install git

```

## 1.2 配置本地用户

```
[root@nfs ~]# git config --global user.name "shaoxh"

```

## 1.3 配置本地的邮箱

```
[root@nfs ~]# git config --global user.email "958678109@qq.com"

```

## 1.4 生成免密钥key

```
[root@nfs ~]# ssh-keygen -t rsa -C 958678109@qq.com

```

## 1.5 把公钥复制给自己的GitHub账号

```
http://www.github.com
Settings
SSH and GPG keys
```

## 1.6 测试连通性

```
[root@nfs ~]# ssh -T git@github.com
```

## 1.7 最后生成三个文件

```
[root@nfs ~]# ls .ssh/
id_rsa  id_rsa.pub  known_hosts

```

## 1.8 本地创建项目

```
[root@nfs ~]# mkdir /mygit
[root@nfs ~]# cd /mygit
[root@nfs mygit]# git init

```

## 1.9 创建远程项目

```
your profile
Repositories
new
Repository name : mygitremote
Public
Create repository
git@github.com:shaoxianheng/mygitremote.git
```

## 2.1 本地项目和远程项目关联

```
git remote add origin
[root@nfs mygit]# git remote add origin  git@github.com:shaoxianheng/mygitremote.git
[root@nfs mygit]#

```

## 2.2 当前文件提交给暂存区

```
[root@nfs mygit]# ll -a
total 4
drwxr-xr-x.  3 root root  29 Mar 12 11:19 .
dr-xr-xr-x. 19 root root 249 Mar 12 09:36 ..
drwxr-xr-x.  7 root root 119 Mar 12 12:21 .git
-rw-r--r--.  1 root root   7 Mar 12 11:19 txt

[root@nfs mygit]# git add .

```

## 2.3 将暂存区提交给当前分支

```
[root@nfs mygit]# git commit -m "第一次发布的内容"

```

## 2.4 将本地分支推送到远程

```
[root@nfs mygit]#  git push -u origin master

```

## 2.5 下载项目

```
[root@nfs ~]# git clone git@github.com:shaoxianheng/mygitremote.git

```

## 2.6 提交 (本地-远程)

```
git add .
git commit -m "提交到分支"
git push  origin master

[root@nfs mygit]# echo shaoxh > test1
[root@nfs mygit]# git add .
[root@nfs mygit]# git commit -m "提交到分支"


```

## 2.7 更新(远程-本地)

```
修改一下远程内容后，操作
[root@nfs mygit]# git pull
[root@nfs mygit]# cat test1
shaoxh
hello world

```

## 2.8 Egit使用

```
Ggit： 在Eclipse 中操作git
目前的eclipse 基本都支持git， 如果不支持，则到eclplise marktplace 搜git 安装配置：
a: window -- Team --git --configuration 
b: preferences -- Network connertions --ssh2
第一次发布
选择项目 -- team -- share Project -- Git --Create --Finish
带有问号的项目存入暂存区 右键 --Team --Add to index
提交到本地分支 右键 -- Team -- Commit --“发布项目”  -- Commit 
将项目推送到远程 右键 -- Team Remote --Push --URI:https://github.com/shaoxianheng/eclipsegitremote.git  (先创建好远程项目)
Protocol：https
user：shaoxianheng
Password：shaoxh8086
Next
Source ref: master  --Add Spec
Next
Finish

第一次下载
file -- import --git --clone URI --NEXT --
更新后的下载： Team -- Repository --Pull

提交
team -- Add to Index
team -- Commit
team -- Commit push

commit时：
commit and　push 或 commit 按钮的区别：
commit按钮： 不能单独的Push 某一个文件， 只能Push 整个项目
commit and push: 可以 单独push 某一个文件

git冲突的解决：
进入同步试图：Team -- Synchronize Workspace
红色就是冲突问题
解决：
  添加到本地暂存区 右键 -- add to index
  提交到本地分支 commit
  更新服务端的分支内容 到本地分支 team -- pull
  修改冲突： 直接修改 或者 右键 -- Merge Tool
                 -- 已经变为了普通本地文件
  add to index
  commit push
```

## 2.9 windos 安装git

```
msysgit.github.io
安装时：Use git from git bash only... 其他默认下一步
配置path： C:\Program Files\Git\bin

进入相应的目录执行 右键连接git bash


```

## 3.1 使用github团队协作

```
远程项目 -- settings
增加合作者 ： Collaborators 加入 合作者： github 全名或者邮箱

发送邀请链接

合作伙伴： 打开该链接、接收邀请 、clone项目 、修改、add \commit \push

```

## 3.2  Git 基础概念

```
git：分布式版本控制器
https：//git-scm.com
Linux系统 --》 BitKeepper（2005收费)
Linux 系统 --》 Git
版本控制系统： 集群版本控制（CVS、svn） 分布式版本控制（git）
git优势：
       本地版本控制	   重写提交说明
svn：  a.txt  “这是我的文件”
git	   a.txt  "这是我的文件" ==》 a.txt "这是我的第一文件"

会议系统： 性能高  (稳定性次要)
         稳定性高 (性能次要)
         
svn：增量
git：全量 （每一个版本都包含全部的文件，时刻保持数据的完整性）
git三种状态(四种)
已修改   （modfied）
已暂存    (staged)
已提交    (commited)

      本机                           远程
工作区 暂存区   对象区                 github服务器
    add --> commit    --> push
第四种是脱离git管理的
版本库（version） ：暂存区+对象区

将某个目录纳入git管理： git init (默认master)
.git：git版本控制的目录


```

## 3.3  查看git的状态

```
[root@localhost gitremote]# git status
[root@localhost gitremote]# git add f1.txt    加入暂存区
[root@localhost gitremote]# git status

```

## 3.4 暂存区还原到工作区

```
[root@localhost gitremote]# git reset HEAD f1.txt
[root@localhost gitremote]# git status

```

## 3.5 暂存区commit 对象区

```
[root@localhost gitremote]# git commit -m "我的内容" f1.txt

```

## 3.6 查看提交的日志

```
[root@localhost gitremote]# git log
commit b72fcb8d13f9b86e6736a465085140a5909732ad
sha1 计算的结果
分布式id生成器

git log  -2 最近两次的提交日志
[root@localhost gitremote]# git log -2 --pretty=oneline
[root@localhost gitremote]# git log -2 --pretty=format:"%h - %an,%ar :%s"
0c960de - shaoxh,2 minutes ago :aa
bb1c724 - shaoxh,9 minutes ago :我的内容

```

## 3.7 git 账号设置 

```
设置邮箱、用户名：
git config --global (基本不用，给整个计算机一次性设置)
git confi --system (推荐，给当前用户一次性设置)
git config --local (给当前项目一次性设置)

提交问题：
       只对修改之后的提交有效，修改之前的提交仍然使用的是之前的配置（用户名、邮箱） 双引号可省略
[root@localhost .git]# git config --global user.name 'shaoxh1'
[root@localhost .git]# git config --global user.email  '958678109@qq.com'
[root@localhost .git]# cat ~/.gitconfig
[user]
        name = shaoxh1
        email = 958678109@qq.com

[root@localhost .git]# git config --system user.name 'shaoxh2'
[root@localhost .git]# git config --system user.email '958678109@qq.com'
[root@localhost .git]# cat /etc/gitconfig
[user]
        name = shaoxh2
        email = 958678109@qq.com

[root@localhost .git]# git config --local user.name 'shaoxh3'
[root@localhost .git]# git config --local user.email '958678109@qq.com'
[root@localhost .git]# cat config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[user]
        name = shaoxh3
        email = 958678109@qq.com

```

## 3.8 回滚修改过的文件

```
[root@localhost gitremote]# vim f1.txt
[root@localhost gitremote]# git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   f1.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
[root@localhost gitremote]# git checkout -- f1.txt

```

## 3.9 删除已被提交的文件

```
[root@localhost gitremote]# git rm f1.txt
rm 'f1.txt'
[root@localhost gitremote]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       deleted:    f1.txt
#
从对象区--》暂存区
彻底删除：git commit -m"彻底删除f1.txt”
git rm ：1. 删除 2.暂存区
[root@localhost gitremote]# git commit -m "彻底删除f1.txt"
[master 985f593] 彻底删除f1.txt
 1 file changed, 2 deletions(-)
 delete mode 100644 f1.txt
[root@localhost gitremote]# git status
# On branch master
nothing to commit, working directory clean
[root@localhost gitremote]# ll
total 8
-rw-r--r--. 1 root root  0 Mar 13 09:14 a.txt
-rw-r--r--. 1 root root  0 Mar 13 09:14 b.txxt
-rw-r--r--. 1 root root 22 Mar 12 20:51 shao
-rw-r--r--. 1 root root 18 Mar 12 20:51 vpn.txt
[root@localhost gitremote]#

操作系统删除： rm ：1. 删除 2.工作区
[root@localhost gitremote]# rm -f a.txt
[root@localhost gitremote]# git add --all
[root@localhost gitremote]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       deleted:    a.txt
#
[root@localhost gitremote]# git commit -m "彻底删除abc"
[master 8e049e0] 彻底删除abc
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 a.txt
[root@localhost gitremote]# git status
# On branch master
nothing to commit, working directory clean


```

## 4.1 后悔删除

```
[root@localhost gitremote]# ll
total 8
-rw-r--r--. 1 root root  0 Mar 13 09:14 b.txxt
-rw-r--r--. 1 root root 22 Mar 12 20:51 shao
-rw-r--r--. 1 root root 18 Mar 12 20:51 vpn.txt
[root@localhost gitremote]# git rm b.txxt
rm 'b.txxt'
[root@localhost gitremote]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       deleted:    b.txxt
#
[root@localhost gitremote]# git reset HEAD b.txxt
Unstaged changes after reset:
D       b.txxt
[root@localhost gitremote]# git status
# On branch master
# Changes not staged for commit:
#   (use "git add/rm <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       deleted:    b.txxt
#
no changes added to commit (use "git add" and/or "git commit -a")
[root@localhost gitremote]# git checkout -- b.txxt
[root@localhost gitremote]# ll
total 8
-rw-r--r--. 1 root root  0 Mar 13 12:16 b.txxt
-rw-r--r--. 1 root root 22 Mar 12 20:51 shao
-rw-r--r--. 1 root root 18 Mar 12 20:51 vpn.txt
[root@localhost gitremote]#

```

## 4.2 重命名操作

```
重命名：move
[root@localhost gitremote]# ll
total 4
-rw-r--r--. 1 root root  0 Mar 13 12:16 b.txxt
-rw-r--r--. 1 root root 22 Mar 12 20:51 shao

[root@localhost gitremote]# git mv shao shao.txt
[root@localhost gitremote]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       renamed:    shao -> shao.txt
#

[root@localhost gitremote]# git reset HEAD shao
[root@localhost gitremote]# git checkout -- shao
[root@localhost gitremote]# ll
total 8
-rw-r--r--. 1 root root  0 Mar 13 12:16 b.txxt
-rw-r--r--. 1 root root 22 Mar 13 22:49 shao
-rw-r--r--. 1 root root 22 Mar 12 20:51 shao.txt

[root@localhost gitremote]# git commit -m "new"

第二种重命名方法：
[root@localhost gitremote]# ll
total 8
-rw-r--r--. 1 root root  0 Mar 13 12:16 b.txxt
-rw-r--r--. 1 root root 22 Mar 13 22:49 shao
-rw-r--r--. 1 root root 22 Mar 12 20:51 shao.txt

[root@localhost gitremote]# mv b.txxt b.txt
[root@localhost gitremote]# git status

[root@localhost gitremote]# git add --all
[root@localhost gitremote]# git commit -m "all"

[root@localhost gitremote]# ll
total 8
-rw-r--r--. 1 root root  0 Mar 13 12:16 b.txt
-rw-r--r--. 1 root root 22 Mar 13 22:49 shao
-rw-r--r--. 1 root root 22 Mar 12 20:51 shao.txt


```

## 4.3 注释重写

```
[root@localhost gitremote]# git commit --amend -m "修正"

```

## 4.4忽略文件

```
gitignore
[root@localhost gitremote]# echo hello > a.properties
[root@localhost gitremote]# touch .gitignore     必须创建此文件
[root@localhost gitremote]# echo a.properties >.gitignore   把文件名写入忽略文件中
[root@localhost gitremote]# git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       .gitignore
nothing added to commit but untracked files present (use "git add" to track)

```

## 4.5 通配符

```
* 任意字符
*.properties

全部的 除去b的
[root@localhost gitremote]# cat .gitignore
*.properties
!b.properties

[root@localhost gitremote]# cp a.properties b.properties
[root@localhost gitremote]# cp a.properties c.properties
[root@localhost gitremote]# vim .gitignore
[root@localhost gitremote]# git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       .gitignore
#       b.properties
nothing added to commit but untracked files present (use "git add" to track)
[root@localhost gitremote]# git commit -m "gitignore"

dir/ 忽略dir目录中的所有文件
dir/*.txt 
dir/*/*.txt
dir/**/*.txt 任意级别目录
创建一个目录
[root@localhost gitremote]# mkdir dir
[root@localhost gitremote]# cp /etc/issue dir/
[root@localhost gitremote]# cp /etc/fstab dir/
[root@localhost gitremote]# git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       dir/
nothing added to commit but untracked files present (use "git add" to track)

[root@localhost gitremote]# cat .gitignore
#*.properties
#!b.properties
dir/

空目录： 默认忽略


```

## 4.6 分支操作

```
查看分支
[root@localhost gitremote]# git branch
* master

创建分支
[root@localhost gitremote]# git branch new_branch

切换分支
[root@localhost gitremote]# git checkout new_branch

删除分支
[root@localhost gitremote]# git checkout master
[root@localhost gitremote]# git branch -d new_branch

创建新分支 并切换： 
[root@localhost gitremote]# git checkout -b new_branch



```

## 4.7 合并分支

```
合并 merger
[root@localhost gitremote]# git merge master
Updating 5253091..888f96d
Fast-forward
 fstab | 11 +++++++++++
 issue |  1 +
 2 files changed, 12 insertions(+)
 create mode 100644 fstab

```

## 4.8 删除分支

```
删除分支 git branch -d  分支名 （不能删除当前分支)
        其他不能删除的情况：包含 “未合并” 的内容，删除分支之前 建议先合并
强行删除 git bransh -D 分支名
细节：
	如果在分支A中进行了写操作，单此操作局限在工作区中进行（没add commit）。在master 中能够看到该操作，
	如果在分支A中进行了写操作，进行了commit（对象区）则master中无法观察到此文件。
	如果在分支A中进行了写操作，但此操作局限在工作区中进行(没add commit)。删除分支A，是可以成功的。

[root@localhost gitremote]# git checkout -b new2_branch
Switched to a new branch 'new2_branch'
[root@localhost gitremote]# git branch
  master
* new2_branch
  new_branch
[root@localhost gitremote]# ls
fstab  issue  passwd
[root@localhost gitremote]# touch shaoxh
[root@localhost gitremote]# git checkout master
Switched to branch 'master'
[root@localhost gitremote]# ll
total 12
-rw-r--r--. 1 root root 465 Mar 14 01:39 fstab
-rw-r--r--. 1 root root  30 Mar 14 01:39 issue
-rw-r--r--. 1 root root 882 Mar 14 01:37 passwd
-rw-r--r--. 1 root root   0 Mar 14 03:22 shaoxh

切换分支
[root@localhost gitremote]# git checkout new2_branch
[root@localhost gitremote]# git add .
[root@localhost gitremote]# git commit -m "提交"

[root@localhost gitremote]# git checkout master
Switched to branch 'master'
[root@localhost gitremote]# ll
total 12
-rw-r--r--. 1 root root 465 Mar 14 01:39 fstab
-rw-r--r--. 1 root root  30 Mar 14 01:39 issue
-rw-r--r--. 1 root root 882 Mar 14 01:37 passwd


```

## 4.9 查看所有分支最近一次提交的信息

```
[root@localhost gitremote]# git branch -v
* master     888f96d ooo
  new_branch 888f96d ooo

```

## 5.1 分支定义

```
分支：一个commit链，一条工作记录线
快照|ersion ：每一次提交后的所有文件  
分支名（master）：指向当前的提交（commit)
HEAD: 指向当前分支 （HEAD -> 分支名）

如果一个分支靠前（dev），另外一个落后（master）.则如果不冲突，master可以通过merge直接追赶上dev：合并，
称之为faster forward

faster forward本质就是 分支指针的移动.注意：跳过的中间commit，仍然会保存。
faster forward: 1.两个分支faster forward 归于一点commit
				2.没有分支信息（丢失分支信息）
git在merge 时，默认使用fast forward； 也可以禁止： git merge --no-ff
				1.两个分支 fast forward， 不会归于一点commit （主动合并的分支 会前进一步）
				2.分支信息完整（不丢失分支信息）

[root@localhost ~]# mkdir /mygit
[root@localhost ~]# cd /mygit
[root@localhost mygit]# ls
[root@localhost mygit]# git init
Initialized empty Git repository in /mygit/.git/
[root@localhost mygit]# touch a.txt

[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m 'init'
[master (root-commit) abc12dc] init
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.txt

创建一个分支dev，查看当前引用的HEAD
[root@localhost mygit]# git checkout -b dev
Switched to a new branch 'dev'
[root@localhost mygit]# cat .git/HEAD
ref: refs/heads/dev

[root@localhost mygit]# git log
commit abc12dcd88b9fb5729422e8e3fc19f8ccc978d0e
Author: shaoxh1 <958678109@qq.com>
Date:   Sun Mar 14 04:03:43 2021 -0400

    init
[root@localhost mygit]# echo 123 > a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "dev1"
[dev 87c4215] dev1
 1 file changed, 1 insertion(+)
[root@localhost mygit]# git log
commit 87c4215ff4dbd0c43de3949397c4f1f511ad3971
Author: shaoxh1 <958678109@qq.com>
Date:   Sun Mar 14 04:08:02 2021 -0400

    dev1

commit abc12dcd88b9fb5729422e8e3fc19f8ccc978d0e
Author: shaoxh1 <958678109@qq.com>
Date:   Sun Mar 14 04:03:43 2021 -0400

    init

dev第二次提交
[root@localhost mygit]#
[root@localhost mygit]# echo abc >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "dev2"
[dev 54d872a] dev2
 1 file changed, 1 insertion(+)

切换master 合并dev
[root@localhost mygit]# git checkout master
Switched to branch 'master'
[root@localhost mygit]# git log
commit abc12dcd88b9fb5729422e8e3fc19f8ccc978d0e
Author: shaoxh1 <958678109@qq.com>
Date:   Sun Mar 14 04:03:43 2021 -0400

    init
[root@localhost mygit]# git merge dev
Updating abc12dc..54d872a
Fast-forward
 a.txt | 2 ++
 1 file changed, 2 insertions(+)
[root@localhost mygit]#
[root@localhost mygit]# git log
commit 54d872a4e985ceb8753cf4af9567db8d8743bb3b
Author: shaoxh1 <958678109@qq.com>
Date:   Sun Mar 14 04:47:18 2021 -0400

    dev2

commit 87c4215ff4dbd0c43de3949397c4f1f511ad3971
Author: shaoxh1 <958678109@qq.com>
Date:   Sun Mar 14 04:08:02 2021 -0400

    dev1

commit abc12dcd88b9fb5729422e8e3fc19f8ccc978d0e
Author: shaoxh1 <958678109@qq.com>
Date:   Sun Mar 14 04:03:43 2021 -0400

    init

```

### --no-ff 

```
[root@localhost mygit]# echo efg >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "dev3"
[dev 4e3cdd7] dev3
 1 file changed, 1 insertion(+)
[root@localhost mygit]# echo 456 >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "dev4"
[dev 9cd76bc] dev4
 1 file changed, 1 insertion(+)
 
 切换master分支
[root@localhost mygit]# git checkout master
[root@localhost mygit]# git merge dev -m "//" --no-ff
[root@localhost mygit]# git log --graph

合并成同步状态
[root@localhost mygit]# git merge master
[root@localhost mygit]# git branch -v
* dev    6959bb8 Merge branch 'dev'
  master 6959bb8 Merge branch 'dev'



```

### 冲突演示

```
[root@localhost mygit]# git checkout master
[root@localhost mygit]# echo master >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "master"

[root@localhost mygit]# git checkout dev
Switched to branch 'dev'
[root@localhost mygit]# echo dev >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "dev"

切换mater分支合并
[root@localhost mygit]# git checkout master
Switched to branch 'master'
[root@localhost mygit]# git merge dev
Auto-merging a.txt
CONFLICT (content): Merge conflict in a.txt
Automatic merge failed; fix conflicts and then commit the result.

解决冲突：
      git add   xxx   git commit -m "xxx"
      git add   xxx   告知git 冲突已解决
      注意：master 在merge 时，如果遇到冲突 并解决，则解决冲突 会进行2此提交： 1次是最终提交，1次是将对方
           的dev的提交信息也拿来了。
[root@localhost mygit]# vim a.txt
[root@localhost mygit]# git branch
  dev
* master
[root@localhost mygit]# cat a.txt
123
abc
efg
456
dev
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "解决冲突"
[master 2621d90] 解决冲突

如果一方落后，另一方前进。则落后方可以直接通过merge合并到前进方。
[root@localhost mygit]# git checkout dev
Switched to branch 'dev'
[root@localhost mygit]# git merge master
Updating de49973..2621d90
Fast-forward
[root@localhost mygit]# git branch -v
* dev    2621d90 解决冲突
  master 2621d90 解决冲突


```

### 显示日志的格式

```
[root@localhost mygit]# git log --graph --pretty=oneline --abbrev-commit
*   2621d90 解决冲突
|\
| * de49973 dev
* | 8e11db9 master
|/
*   6959bb8 Merge branch 'dev'
|\
| * 9cd76bc dev4
| * 4e3cdd7 dev3
|/
* 54d872a dev2
* 87c4215 dev1
* abc12dc init

```

## 5.2 版本穿梭

```
版本穿梭： 在多个commit之间 进行穿梭。回退、前进
合并add 和commit：
git commit -am '注释'
重新创建mygit目录
[root@localhost mygit]# git init
Initialized empty Git repository in /mygit/.git/
[root@localhost mygit]#
[root@localhost mygit]#
[root@localhost mygit]# echo hello master >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "init1"
[master (root-commit) f1488f4] init1
 1 file changed, 1 insertion(+)
 create mode 100644 a.txt
[root@localhost mygit]# echo 222 >> a.txt
[root@localhost mygit]# git commit -am 'init2'
[master d5dec11] init2
 1 file changed, 1 insertion(+)
[root@localhost mygit]# echo 333 >> a.txt
[root@localhost mygit]# git commit -am 'init3'
[master 387bb65] init3
 1 file changed, 1 insertion(+)
[root@localhost mygit]# echo 444 >> a.txt
[root@localhost mygit]# git commit -am 'init4'
[master ef56ded] init4
 1 file changed, 1 insertion(+)
 
##### [root@localhost mygit]# git commit --amend -m "init4"   //修改注释

[root@localhost mygit]# git log|grep commit
commit 63fd37fb3ab4ac94ac9136ce91d4481adefd939d
commit 387bb65b2b8aa98c02c728e267393d3f62a2c270
commit d5dec1170838e6b6d35186fa82df85a3c6b8914d
commit f1488f4b3b550f841eb12f1995f452c6e541ff30

回退到上一次commit：
[root@localhost mygit]# git reset --hard HEAD^
HEAD is now at 387bb65 init3

回退两次：
[root@localhost mygit]# git reset --hard HEAD^^
HEAD is now at f1488f4 init1

回退n次：n代表的是数字
git reset --hard HEAD~n

回退到前任意次：通过sha1 直接回退
git reset --hard sha1的值

```

### reflog查看记录，

```
记录所有操作，可以帮我们 实现 ”后悔“ 操作。需要借助于良好的注释习惯。
[root@localhost mygit]# git reflog
f1488f4 HEAD@{0}: reset: moving to HEAD^^
387bb65 HEAD@{1}: reset: moving to HEAD^
63fd37f HEAD@{2}: commit (amend): init4
9f2ea45 HEAD@{3}: commit (amend): in4
ef56ded HEAD@{4}: commit: init4
387bb65 HEAD@{5}: commit: init3
d5dec11 HEAD@{6}: commit: init2
f1488f4 HEAD@{7}: commit (initial): init1

[root@localhost mygit]# git reset --hard 9f2e
[root@localhost mygit]# git log |grep commit
commit 9f2ea456ab1a16ec7b97b8fb786268c6f27e89cb
commit 387bb65b2b8aa98c02c728e267393d3f62a2c270
commit d5dec1170838e6b6d35186fa82df85a3c6b8914d
commit f1488f4b3b550f841eb12f1995f452c6e541ff30

```

## 5.3 checkout的放弃与游离操作

```
[root@localhost /]# mkdir /mygit
[root@localhost /]# cd /mygit
[root@localhost mygit]# git init
Initialized empty Git repository in /mygit/.git/
[root@localhost mygit]# echo aaa > a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "init"
[root@localhost mygit]# vim a.txt
添加一行bbb
[root@localhost mygit]# git add .
[root@localhost mygit]# vim a.txt
添加一行ccc
[root@localhost mygit]# git status
[root@localhost mygit]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   a.txt
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   a.txt
#
checkout : 放弃修改。放弃的是工作区的修改，相对于暂存区或对象区

[root@localhost mygit]# git checkout f1.txt
[root@localhost mygit]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   a.txt
[root@localhost mygit]# cat a.txt
aaa
bbb

提交
[root@localhost mygit]# git commit -m "222"

状态
[root@localhost mygit]# git status
# On branch master
nothing to commit, working directory clean

reset: 将之前增加到暂存区中的内容 回退到工作区

添加一行新的数据xxx
[root@localhost mygit]# echo xxx >> a.txt
[root@localhost mygit]# git add a.txt
[root@localhost mygit]# git status
[root@localhost mygit]# git reset HEAD a.txt

checkout：版本穿梭（游离状态）
1. 修改后、必须提交
2. 创建分支的好时机，git branch mybranch 


```

## 5.4 分支重命名

```
[root@localhost mygit]# git branch -m master master2

```

## 5.5 stash 保存现场

```
1. 建议（规范）：在功能未没有开发完毕前，不要commit
2. 规定（必须）：在没有commit之前，不能checkout 切换分支

如果还没有将某一个功能开发完毕，就要切换分支，
建议 1. 保存现场（临时保存、stash）
	2.切换

创建一个dev分支
[root@localhost mygit]# git checkout -b dev
[root@localhost mygit]# echo hello dev >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "dev1"

[root@localhost mygit]# echo hello dev2 >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "dev2"

不提交不能切换
[root@localhost mygit]# echo hello dev3  >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git checkout master
error: Your local changes to the following files would be overwritten by checkout:
        a.txt
Please, commit your changes or stash them before you can switch branches.
Aborting

保存现场
[root@localhost mygit]# git stash

切换分支
[root@localhost mygit]# git checkout master
Switched to branch 'master'

查看现场
[root@localhost mygit]# git stash list

还原现场
[root@localhost mygit]# git stash pop   （将原来保存的删除，用于还原内容）
[root@localhost mygit]# git stash apply  (还原内容)


[root@localhost mygit]# echo hello dev4  >> a.txt
[root@localhost mygit]# git stash

[root@localhost mygit]# echo hello dev5 >> a.txt
[root@localhost mygit]# git stash save "mystash"
Saved working directory and index state On dev: mystash
HEAD is now at e045940 dev2
[root@localhost mygit]# git stash list
stash@{0}: On dev: mystash
stash@{1}: WIP on dev: e045940 dev2

还原现场
（默认还原最近一次）
git stash pop （将原来保存的删除，用户还原内容)
git stash apply (还原内容，不删除原保存的内容)  可以指定某一次现场 git stash apply stash@{1}
git stash drop stash@{0}  (手工删除现场)    
[root@localhost mygit]# git stash apply

如果不同的分支，在同一个commit阶段，在commit之前，可以checkout切换分支



```

## 5.6 标签

```
Tag标签：适用于整个项目，和具体的分支没有关系
git tag xxx
git tag -a xxx -m "xxx"
查看标签 git tag
删除标签 git tag -d 标签名

[root@localhost mygit]# git checkout dev
Switched to branch 'dev'
[root@localhost mygit]# echo tag >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "first tag"
[dev d3c06cd] first tag
 1 file changed, 1 insertion(+)
[root@localhost mygit]# git tag v1.0
[root@localhost mygit]# git tag
v1.0
[root@localhost mygit]#


[root@localhost mygit]# echo tag2 >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "sencond ta"
[dev 88fac89] sencond ta
 1 file changed, 1 insertion(+)
[root@localhost mygit]# git tag -a v2.0 -m "release Tag"
[root@localhost mygit]# git tag
v1.0
v2.0
[root@localhost mygit]#

删除标签
[root@localhost mygit]# git tag -d v1.0
Deleted tag 'v1.0' (was d3c06cd)
[root@localhost mygit]# git tag
v2.0

模糊查询
[root@localhost mygit]# git tag -l 'v*'
v2.0

[root@localhost mygit]# git checkout master
Switched to branch 'master'
[root@localhost mygit]# git tag
v2.0
[root@localhost mygit]# git tag -d 'v2.0'
Deleted tag 'v2.0' (was c2fd0b0)


```

## 5.6 blame ：责任

```
git blame a.txt 查看a.txt 的所有提交commit sha1 值，以及每一行的作者
[root@localhost mygit]# git blame a.txt
564f0d15 (shaoxh 2021-03-21 21:49:47 -0400 1) aaa
081f2641 (shaoxh 2021-03-21 22:07:17 -0400 2) bbb
42a68ff4 (shaoxh 2021-03-21 22:08:29 -0400 3) ccc
ec8a5efc (shaoxh 2021-03-21 22:42:14 -0400 4) ddd
f1010505 (shaoxh 2021-03-21 23:18:52 -0400 5) hello dev
e045940a (shaoxh 2021-03-21 23:19:56 -0400 6) hello dev2
8a38d85c (shaoxh 2021-03-22 01:13:44 -0400 7) hello dev3
8a38d85c (shaoxh 2021-03-22 01:13:44 -0400 8) hello dev4

```

## 5.7 diff 差异性

```
[root@localhost mygit]# rm -rf *
[root@localhost mygit]# git init
[root@localhost mygit]# echo first line ... > a.txt
Reinitialized existing Git repository in /mygit/.git/
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "init"
[master cd498e2] init
 1 file changed, 1 insertion(+), 8 deletions(-)
 
[root@localhost mygit]# echo second line >> a.txt
[root@localhost mygit]# git add .

[root@localhost mygit]# echo third line ... >> a.txt
[root@localhost mygit]# git diff
git diff： 比较的区中的文件，暂存区和工作区的差异

比较对象区和工作区的差异
[root@localhost mygit]# rm -rf *
[root@localhost mygit]# echo 1 >> a.txt
[root@localhost mygit]# git init
Reinitialized existing Git repository in /mygit/.git/
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "1"
[master 50e7ac9] 1
 1 file changed, 1 insertion(+), 1 deletion(-)
[root@localhost mygit]# echo 2 >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m "2"
[master 4a140fa] 2
 1 file changed, 1 insertion(+)
[root@localhost mygit]# echo 3 >> a.txt
[root@localhost mygit]# git diff
diff --git a/a.txt b/a.txt
index 1191247..01e79c3 100644
--- a/a.txt
+++ b/a.txt
@@ -1,2 +1,3 @@
 1
 2
+3

比较对象区和暂存区的差异
[root@localhost mygit]# git add .
[root@localhost mygit]# git diff --cached
diff --git a/a.txt b/a.txt
index 1191247..01e79c3 100644
--- a/a.txt
+++ b/a.txt
@@ -1,2 +1,3 @@
 1
 2
+3



```

## 5.8 github

```
push: 本地 --》 github
pull： github --》 本地  pull = fetch + merge
rm -rf * ：不会删除隐藏文件
github的仓库中，默认的说明文档README.md

推送：
git remote add origin git@github.com:shaoxianheng/mygit.git
git push -u origin master

后续修改推送时，只需要git push

ssh 配置： 本地存放私钥， 远程github存放公钥
ssh-keygen  生成： 私钥(本机)  公钥(github)
可以将公钥存放在github中的两个地方
    项目的setting 中，只要当前项目可以和本机免秘钥登录
    账号的setting 中，账号的所有项目都可以和本机免秘钥
注意： 远程增加ssh的公钥时， 1删除回车符 2 添加可写权限
   
   dev： 开发分支，频繁改变
   test: 基本开发完毕后，交给测试实施人员的分支  
   master:  生产阶段，很少变化
   bugfix： 临时修复bug分支
 
 dev --> test (merge dev) --> master (merge test) --> ...
 
 查看服务器列表的详细信息
[root@localhost mygit]# git remote show origin

查看当前服务器的列表
[root@localhost mygit]# git remote show
origin

git 会在本地维护 origin/master 分支，通过该分支 感知远程GitHub的内容

[root@localhost mygit]# git branch -a
* master
  remotes/origin/master
  
[root@localhost mygit]# git branch -av
* master                d2531e0 111
  remotes/origin/master d2531e0 111
[root@localhost mygit]# echo shaoxh >> a.txt
[root@localhost mygit]# git add .
[root@localhost mygit]# git commit -m 'sxh'
[master 3204858] sxh
 1 file changed, 1 insertion(+)
[root@localhost mygit]# git branch -av
* master                3204858 [ahead 1] sxh
  remotes/origin/master d2531e0 111
[root@localhost mygit]# git status
# On branch master
# Your branch is ahead of 'origin/master' by 1 commit.
#   (use "git push" to publish your local commits)
#
nothing to commit, working directory clean
[root@localhost mygit]# git push

[root@localhost mygit]# git branch -av
* master                3204858 sxh
  remotes/origin/master 3204858 sxh


```



## 5.9 本地与远程冲突问题

```
pull /push ：推送，改变指针
fast-forward : 更新，如果发现更新的内容比自己先一步（commit的sha1值在自己之前），则会自动合并。

冲突: 
```

