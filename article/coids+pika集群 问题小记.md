#coids+pika集群 问题小记
# 概要

近期搭建coids+pika集群，碰到一些小坑，此文记录下流程以及碰到的问题。

>  
 pika github： codis github： 


碰到问题的时候第一时间除了google之外，其实也可以去官方github的issue中搜索下，因为一般流行框架的坑都会有人先踩过了。 如果你发现你的问题google和官网github 的issue上都找不到，那么首先要怀疑下你自己的操作，其次再去怀疑服务器环境是不是有问题。当然，大部分的问题都是可以根据报错来直接找到解决办法的。

# 主要流程

1、搭建codis-dashboard 和codis-fe 服务器 2、搭建pika 服务器(需要是pika支持codis的版本) 3、搭建codis-proxy 服务器 4、codis-fe添加 codis-proxy和pika 服务器，并且均衡slot。

# 主要部署方式

1、安装go环境+git clone codis代码+编译 2、安装docker+安装运行镜像

个人推荐使用docker的方式部署。首先安装go环境会比较麻烦，另外系统环境等原因很容易影响codis的编译，不同服务器上遇到的问题可能还会不一样。

# 问题

## centos 安装docker-ce

```
Error: Package: 3:docker-ce-18.09.1-3.el7.x86_64 (docker-ce-stable)
           Requires: container-selinux &gt;= 2.9
Error: Package: 3:docker-ce-18.09.1-3.el7.x86_64 (docker-ce-stable)
           Requires: libseccomp &gt;= 2.3
           Available: libseccomp-2.2.1-1.el7.i686 (CENTOS7.2-basic)
               libseccomp = 2.2.1-1.el7

```

解决：安装container-selinux 2.9

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum install epel-release
yum install container-selinux 

```

## centos docker ：An error occurred trying to connect

```
$ docker ps
An error occurred trying to connect: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json: read unix @-&gt;/var/run/docker.sock: read: connection reset by peer

```

解决：yum更新，重装docker-engine

```
yum remove docker-engine
yum update
yum install docker-engine

```

## pika 编译不同过问题

解决：可以直接下载pika的realease包，解压后直接使用。

## pika 3.0以上暂时不支持coids（现在pika更新到3.0.8版本）

pika 3.0以上版本找不到slotmigrate这个变量，codis提示如下：

```
ERR unknown command 'SLOTSINFO

```

解决：可以暂时使用pika 2.3.6版本。

## 服务器外网网速太慢

使用SFTP，先将文件下载到自己的机子，然后传到对应的服务器。 （在笔者使用的时候传输了docker的镜像，包括pika的release包）
