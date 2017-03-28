---
layout: post
title: "使用Harbor+Ceph在单机搭建docker镜像管理"
date: 2017-03-28
comments: true
category: docker
tags: [docker, ceph, harbor]
---

## 1.背景

因为公司需要, 要前台使用Harbor作为docker镜像管理系统,后台使用Ceph作为存储

Harbor地址: https://github.com/vmware/harbor  
Ceph地址:http://docs.ceph.com/docs/jewel/

因为是测试,所以全部在单机下安装.  

## 2. 环境准备

因为Ceph官网主要针对多机环境进行的安装,单机环境需要进行一些调整  

环境准备: Centos 7服务器一台

## 3. 安装Harbor

Harbor的安装比较简单,安装地址(https://github.com/vmware/harbor/blob/master/docs/installation_guide.md)已经写得比较详细了

这个地方只要是安装文档一步一步来是没有问题的,如果启动错误,注意看下容器的错误日志,如果容器启动成功,运行过程中有问题可以去默认日志`/var/log/harbor`查找对应模块的错误,如果还有问题可以发邮件给我

## 4. Ceph的安装情况说明

我曾经在这个地方走过一个弯路,曾经想使用docker安装,ceph-docker地址(https://hub.docker.com/r/ceph/daemon/), github地址(https://github.com/ceph/ceph-docker)

最后验证拉取镜像出现以下错误:

```
bj-m-xxxxxxa:~ xxxxxxa$ docker pull {IPaddress}:6000/cetos:v7
v7: Pulling from cetos
4969bbd91a1e: Retrying in 5 seconds
error pulling image configuration: unexpected EOF
```

这个原因我查了好久,没有明显的错误日志,随放弃转成实体机安装,实体机安装成功,我现在怀疑是不是ceph/damon这个镜像存在问题

## 5.Ceph安装步骤

由于是单机,没有像官网安装那样配置无密码访问,所有操作在root下进行,官网安装地址:http://docs.ceph.com/docs/jewel/start/

### 添加Ceph源

```
sudo vim /etc/yum.repos.d/ceph-deploy.repo
```

这个地方注意,官网的`sudo vim /etc/yum.repos.d/ceph.repo`,但是很多出现以下错误:

```
[ceph-admin][WARNIN] ensuring that /etc/yum.repos.d/ceph.repo contains a high priority
[ceph_deploy][ERROR ] RuntimeError: NoSectionError: No section: 'ceph'
```

改成`ceph-deploy.repo`即可解决,

```
sudo mv /etc/yum.repos.d/ceph.repo /etc/yum.repos.d/ceph-deploy.repo
```

至于为什么这样,地址:http://www.virtualtothecore.com/en/adventures-with-ceph-storage-part-5-install-ceph-in-the-lab/ 下面的评论有讨论的,我看下了,最后不知道怎么自动生成了一个`ceph.repo`

ceph-deploy.repo文件内容:

```
[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
```

### 安装ceph-deploy

执行命令:

```
sudo yum makecache
sudo yum install ntp ntpdate ntp-doc  # 这个主要是多机用来时间同步的
sudo yum install ceph-deploy
```

关闭SELINUX

```
sudo setenforce 0
```

确保端口6800:7300打开

更改hostname

```
hostname cephnode1
```

更改`/etc/hosts`,添加:

```
cephnode1  xxx.xxx.xxx.xxx(本机IP地址)
```

### 安装组件

1. 创建文件夹

    ```
    mkdir my-cluster
    cd my-cluster
    ```
2. 创建一个集群

    ```
    ceph-deploy new cephnode1 # cephnode1就是设置的hostname名称
    ```

3. 这时候当前文件夹下应该生成了`ceph.conf`配置文件,修改配置文件,以下是需要添加的:

    ```
    osd pool default size = 2
    osd pool default min size =1
    public network = 10.0.0.0/24
    cluster network = 10.0.0.0/24
    osd crush chooseleaf type = 0  # 单机需要添加
    ```
    
4. 初始化mon

    ```
    ceph-deploy mon create-initial
    ```
    
5. 准备和激活OSDS, 我这里初始化了三个OSD

    ```
    ceph-deploy osd prepare \
    cephnode1:/ceph/sdb1 cephnode1:/ceph/sdb2 cephnode1:/ceph/sdb3
    ```
    
    `/ceph/sdb1`, `/ceph/sdb2`, `/ceph/sdb3`挂载了三个分区:
    
    ```
    [root@cephnode1 my-cluster]# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda3       3.9G  299M  3.4G   9% /
    devtmpfs        5.8G     0  5.8G   0% /dev
    tmpfs           5.8G     0  5.8G   0% /dev/shm
    tmpfs           5.8G  570M  5.3G  10% /run
    tmpfs           5.8G     0  5.8G   0% /sys/fs/cgroup
    /dev/sda1        12G  3.4G  7.8G  30% /usr
    /dev/sda5       7.8G   38M  7.3G   1% /tmp
    /dev/sda6       7.8G  3.9G  3.6G  53% /var
    /dev/sda7        33G  5.1G   28G  16% /ceph/sdb1  # 用于ceph
    /dev/sda8        33G  5.1G   28G  16% /ceph/sdb2  # 用于ceph
    /dev/sda9        33G  5.1G   28G  16% /ceph/sdb3  # 用于ceph
    tmpfs           1.2G     0  1.2G   0% /run/user/0
    ```
    
    激活osd:
    
    ```
    ceph-deploy osd activate \
    cephnode1:/ceph/sdb1 cephnode1:/ceph/sdb2 cephnode1:/ceph/sdb3
    ```

6. 部署rgw

    ```
    ceph-deploy rgw create cephnode1
    ```
    
    记录下rgw监听的端口hao,默认:7480
    
7. 创建Ceph Swift API

   ```
   radosgw-admin user create --uid=registry --display-name="registry" 
   radosgw-admin subuser create --uid=registry --subuser=registry:swift --access=full 
   ```
   
   记录下这两天命令生成的JSON文本,连接registry需要用,尤其是第二个命令生成的`swift_keys`:
   
   ```
       "swift_keys": [
        {
            "user": "registry:swift",
            "secret_key": "Qntoa1n3lRxjnP1iU5cvjMpUSOczDnasc0wwI6V1"
        }
    ]
   ```
   
### 执行harbor与ceph对接

如果harbor还没有安装,需要到harbor启动文件`common/templates/registry/config.yml`修改,然后执行'install.sh'进行安装

如果已经安装了,那么直接修改`/common/config/registry/config.yml`,然后重启即可

主要修改config.yml的storage部分:

```
storage:
    cache:
        layerinfo: inmemory
    swift:
       authurl: http://10.0.0.3:7480/auth/v1
       username: registry:swift
       password: Qntoa1n3lRxjnP1iU5cvjMpUSOczDnasc0wwI6V1
       container: registry
    maintenance:
        uploadpurging:
            enabled: false
    delete:
        enabled: true
```

重启harbor,测试搭建是否成功.
