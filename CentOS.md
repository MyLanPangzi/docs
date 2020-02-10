# CentOS

## 修改yum源

1. 修改为163源

   ```shell
   mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
   #记得改系统Centos-678
   wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
   yum clean all
   yum makecache
   ```

## 安装MySQL

1. https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html

```
yum install mysql-server.x86_64 -y
systemctl start mysqld
mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Xiebo0409';

```

