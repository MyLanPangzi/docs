# CentOS

## 防火墙

## 网卡

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
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Xiebo0409';

```

## Docker

```shell
sudo yum  remove podman-manpages.noarch -y
#https://download.docker.com/linux/centos/7/x86_64/stable/Packages/
#下载最新的 containerd docker-ce-cli docker-ce 
#下载libcgroup
#rpm -ivh --nodeps --force
```

