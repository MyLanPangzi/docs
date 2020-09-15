[Parent](../README.md)

# CentOS

## 防火墙
```shell script
systemctl disable --now firewalld
```

## 关闭警告

```shell script
vim /etc/inputrc
set bell-style visible
```

## 网卡

```shell script
vim /etc/sysconfig/network-scripts/ifcfg-ens33
systemctl restart NetworkManager

DEVICE=ens33
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=static
NAME="ens33"
PREFIX=24
IPADDR=192.168.1.102
GATEWAY=192.168.1.2
DNS1=192.168.1.2

```

## 超级用户无密码

```shell script
vi /etc/sudoers
# 修改/etc/sudoers文件，找到下面一行（102行），在%wheel下面添加一行：
histcat   ALL=(ALL)     NOPASSWD: ALL
```

## 配置hosts

```shell script
vi /etc/hosts

192.168.242.102 hadoop102
192.168.242.103 hadoop103
192.168.242.104 hadoop104
```

## 配置SSH

```shell script
ssh-keygen -t rsa
ssh-copy-id hadoop102
ssh-copy-id hadoop103
ssh-copy-id hadoop104
```

## 同步脚本

```shell script
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
  echo Not Enough Arguement!
  exit;
fi
#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
  echo ====================  $host  ====================
  #3. 遍历所有目录，挨个发送
  for file in $@
  do
    #4 判断文件是否存在
    if [ -e $file ]
    then
      #5. 获取父目录
      pdir=$(cd -P $(dirname $file); pwd)
      #6. 获取当前文件的名称
      fname=$(basename $file)
      ssh $host "mkdir -p $pdir"
      rsync -av $pdir/$fname $host:$pdir
    else
      echo $file does not exists!
    fi
  done
done
```

## 远程执行脚本

```shell script
#!/usr/bin/env bash

for i in hadoop102 hadoop103 hadoop104 ; do
    ssh -t $i "$@"
done

```

## 内网穿透

## Yum源

1. 修改为163源

   ```shell
   mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
   #记得改系统Centos-678
   wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
   yum clean all
   yum makecache
   ```

## MySQL

1. https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html

```
yum install mysql-server.x86_64 -y
systemctl start mysqld
mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY '密码';

```

## Docker

```shell
sudo yum  remove podman-manpages.noarch -y
#https://download.docker.com/linux/centos/7/x86_64/stable/Packages/
#下载最新的 containerd docker-ce-cli docker-ce 
#下载libcgroup
#rpm -ivh --nodeps --force
```

