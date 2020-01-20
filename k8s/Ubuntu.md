# Ubuntu常见配置

## Apt-get源

```shell
cp /etc/apt/sources.list /etc/apt/sources.list.bak
vim /etc/apt/sources.list
#:%s/cn.archive.ubuntu.com/mirrors.aliyun.com
```



## 防火墙

```shell
ufw disable
```

## 交换空间

```shell
swapoff a
#永久关闭，注释掉第一行
vim /et/fstab
```



## Netplan（配置静态IP）

```shell
vim /etc/netplan/50-cloud-init.yaml
netplan apply
```

```yml
network:
  ethernets:
    ens33: #注意配置网卡
      addresses: [192.168.11.138/24]
      gateway4: 192.168.11.2
      nameservers:
        addresses: [114.114.114.114, 192.168.11.2]
      dhcp4: no
      optional: no
  version: 2
```

## DNS

```shell
vim /etc/systemd/resolve.conf
[Resolve]
DNS=114.114.114.144
#FallbackDNS=
#Domains=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#Cache=yes
#DNSStubListener=yes

```

## Docker

```shell
apt-get install -y docker.io
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://a17tqp4p.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Kubernetes

```shell
sudo apt update && sudo apt install -y apt-transport-https curl
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" >>/etc/apt/sources.list.d/kubernetes.list
sudo apt update && apt install -y kubelet kubeadm kubectl && apt-mark hold kubelet kubeadm kubectl
```

## 同步时区

```shell
dpkg-reconfigure tzdata
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 同步时间

```shell
sudo apt-get install ntpdate && ntpdate cn.pool.ntp.org && hwclock --systohc
```

## 主机名

```shell
hostnamectl set-hostname ubuntu18-server
#hostnamectl set-hostname ubuntu18-server-master
#hostnamectl set-hostname ubuntu18-server-node1
#hostnamectl set-hostname ubuntu18-server-node2
vim /etc/cloud/cloud.cfg

#preserve_hostname: true
```

## 扩容

```shell
#查看磁盘使用情况
df -h
#查看卷组使用情况
vgdisplay
#分配磁盘
lvextend -l 50G /dev/mapper/ubuntu1604--vg-root
lvextend -l +100%FREE /dev/mapper/ubuntu1604--vg-root
#重新计算大小
resize2fs  /dev/mapper/ubuntu1604--vg-root
#查看磁盘使用情况
df -h
vgdisplay
```

## 安装NFS服务

```shell
#先关闭防火墙
apt-get update && apt install nfs-kernel-server -y
mkdir -p /mnt/sharedfolder
chown nobody:nogroup /mnt/sharedfolder
chmod 777 /mnt/sharedfolder
vim /etc/exports
#/mnt/sharedfolder 192.168.2.0/24(rw,sync,no_subtree_check)
exportfs -a
systemctl restart nfs-kernel-server
#ufw allow from [clientIP or clientSubnetIP] to any port nfs
#ufw allow from 192.168.100/24 to any port nfs
ufw status
apt-get update && apt-get install -y nfs-common nfs-utils
```

## 安装NFS客户端

```shell
apt-get update && apt-get install nfs-common -y
mkdir -p /mnt/sharedfolder_client
mount 192.168.2.40:/mnt/sharedfolder /mnt/sharedfolder_client

```

