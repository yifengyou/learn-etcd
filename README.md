<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [learn-etcd](#learn-etcd)   
   - [相关站点](#相关站点)   
   - [etcd介绍](#etcd介绍)   
   - [基本操作](#基本操作)   
      - [安装](#安装)   
      - [配置](#配置)   
      - [获取版本](#获取版本)   
      - [创建键值C](#创建键值c)   
      - [根据键获取值R](#根据键获取值r)   
      - [更新键值U](#更新键值u)   
      - [根据键删除D](#根据键删除d)   
      - [监听键的变动](#监听键的变动)   
      - [向集群中新加节点](#向集群中新加节点)   
      - [查看集群中存在的节点](#查看集群中存在的节点)   
      - [删除集群中的节点](#删除集群中的节点)   
      - [查看健康度](#查看健康度)   

<!-- /MDTOC -->

# learn-etcd

![20210722_165545_70](image/20210722_165545_70.png)

```
Something I hope you know before go into the coding~
First, please watch or star this repo, I'll be more happy if you follow me.
Bug report, questions and discussion are welcome, you can post an issue or pull a request.
```

## 相关站点

* <https://github.com/etcd-io/etcd>

## etcd介绍

* etcd是CoreOS团队于2013年6月发起的开源项目，它的目标是构建一个高可用的分布式键值(key-value)数据库。
* etcd内部采用raft协议作为一致性算法
* etcd基于Go语言实现

特点：

1. 简单：安装配置简单，而且提供了HTTP API进行交互，使用也很简单
2. 安全：支持SSL证书验证
3. 快速：根据官方提供的benchmark数据，单实例支持每秒2k+读操作
4. 可靠：采用raft算法，实现分布式系统数据的可用性和一致性


## 基本操作

### 安装

```
sudo yum install -y etcd
sudo systemctl enable etcd --now
```

配置文件位于:```/etc/etcd/etcd.conf```

### 配置


默认配置为：
```
cat /etc/etcd/etcd.conf |grep -vE '^#|^$'
ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"
```

执行下列命令：

```
sudo mkdir -p /data/etcd
sudo chown etcd:etcd /data/etcd
sed -i 's/^ETCD_NAME=.*/ETCD_NAME=myetcd/g' /etc/etcd/etcd.conf
sed -i 's/^ETCD_LISTEN_CLIENT_URLS.*/ETCD_LISTEN_CLIENT_URLS="http:\/\/0.0.0.0:2379"/g' /etc/etcd/etcd.conf
sed -i 's/^ETCD_DATA_DIR.*/ETCD_DATA_DIR="\/data\/etcd"/g' /etc/etcd/etcd.conf
sed -i 's/^ETCD_ADVERTISE_CLIENT_URLS.*/ETCD_ADVERTISE_CLIENT_URLS="http:\/\/0.0.0.0:2379"/g' /etc/etcd/etcd.conf
```

修改为：

```
ETCD_NAME=myetcd
ETCD_DATA_DIR="/data/etcd"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379"
```

重启服务

```
systemctl daemon-reload
systemctl restart etcd
```

### 获取版本

```
[root@vm ~]# etcd --version
etcd Version: 3.4.14
Git SHA: Not provided (use ./build instead of go build)
Go Version: go1.16beta1
Go OS/Arch: linux/amd64
```

### 创建键值C

* etcd在键的组织上采用了层次化的空间结构(类似于文件系统中目录的概念)
* 用户指定的键可以为单独的名字，如:testkey，此时实际上放在根目录/下面，
* 可以为指定目录结构，如/cluster1/node2/testkey，则将创建相应的目录结构
* 数据库操作围绕对键值和目录的CRUD（Create,Read,Update,Delete）完整生命周期的管理。

```
etcdctl put /helloetcd/firstkey "Hello world"
```

### 根据键获取值R

```
etcdctl get /helloetcd/firstkey
```

### 更新键值U

```
etcdctl put /helloetcd/firstkey "Hello world etcd"
etcdctl get /helloetcd/firstkey
```

### 根据键删除D

```
etcdctl del /helloetcd/firstkey
etcdctl get /helloetcd/firstkey
```

### 监听键的变动

不会显示R，仅显示U、D、C

```
etcdctl watch /helloetcd/firstkey
```

### 向集群中新加节点

```
[root@vm ~]#  etcdctl member add etcd3 http://localhost:2381
Added member named myetcd2 with ID 8e9e05c53164495d to cluster
```

### 查看集群中存在的节点

```
[root@vm ~]# etcdctl member list
8e9e05c52164694d, started, myetcd, http://localhost:2380, http://0.0.0.0:2379, false
8e9e05c53164495d, started, myetcd2, http://localhost:2381, http://0.0.0.0:2372, false
```

### 删除集群中的节点

```
[root@vm ~]#  etcdctl member remove 8e9e05c53164495d
Removed member 8e9e05c53164495d from cluster
```

### 查看健康度

```
[root@vm ~]#  etcdctl endpoint health
```





---
