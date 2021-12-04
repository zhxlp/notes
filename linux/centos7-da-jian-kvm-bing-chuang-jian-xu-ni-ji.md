# CentOS7 搭建 kvm 并创建虚拟机

## 环境

### 系统版本

```bash
cat /etc/centos-release
```

![](../.gitbook/assets/Snipaste_2018-10-18_14-16-45.png)

### 检查 cpu 是否支持处理器虚拟化

```bash
egrep -o '(vmx|svm)' /proc/cpuinfo
```

![](../.gitbook/assets/Snipaste_2018-10-18_14-02-41.png)

### 检查是否加载 kvm 模块

```bash
lsmod | grep kvm
```

![](../.gitbook/assets/Snipaste_2018-10-18_14-11-55.png)

## 安装步骤

### 安装必要软件

```bash
yum -y install libvirt qemu-kvm virt-install
```

### 启动 libvirt 并设置开机自启

```bash
systemctl start libvirtd.service
systemctl enable libvirtd.service
```

### 配置网桥

查看现有网卡配置文件

```bash
ls /etc/sysconfig/network-scripts | grep ifcfg-*
```

![1539844856994](../.gitbook/assets/1539844856994.png)

备份网卡配置

```bash
cp /etc/sysconfig/network-scripts/ifcfg-ens33 /etc/sysconfig/network-scripts/ifcfg-ens33.bak
```

编辑网卡文件 ifcfg-ens33,将配置文件中的 ip 地址、掩码、网关、DNS 等信息注释，并增加一行 BRIDGE=br0

```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

![1539847045569](../.gitbook/assets/1539847045569.png)

创建并编辑桥接网卡文件 ifcfg-br0

```bash
vi /etc/sysconfig/network-scripts/ifcfg-br0
```

配置内容如下

```bash
TYPE=Bridge
DEVICE=br0
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.49.30
NETMASK=255.255.255.0
GATEWAY=192.168.49.2
DNS1=114.114.114.114
DNS2=8.8.8.8
```

重启网络服务

```bash
systemctl restart network
```

### 创建 kvm 虚拟机

```bash
virt-install --name WindowServer2008R2 \
--vcpus 1 \
--memory 1024 \
--os-variant win2k8r2 \
--disk path=/var/lib/libvirt/images/WindowServer2008R2.qcow2,size=20,format=qcow2,bus=virtio \
--disk path=/tmp/WindowsServer2008.iso,device=cdrom \
--disk path=/tmp/virtio-win-0.1.141.iso,device=cdrom \
--network bridge=br0,model=virtio \
--graphics vnc,listen=0.0.0.0 \
--noautoconsole
```

> --name 虚拟机名字
>
> --vcpus 虚拟机 cpu 数量
>
> --memory 虚拟机内存，单位 MB
>
> --os-variant 虚拟机操作系统类型，可通过命令 ’osinfo-query os‘ 查询系统类型列表
>
> --disk 虚拟机硬盘文件的信息
>
> ​ path 存放路径
>
> ​ size 硬盘大小，单位 GB
>
> ​ format 硬盘格式，可填 raw,qcow,qcow2,vmdk 等等
>
> --cdrom 光驱设置，可用作安装媒介
>
> --network 虚拟机网络
>
> --graphics 图像连接模式，可填 vnc,spice,none
>
> --noautoconsole 创建时，不自动连接到虚拟机

临时放行服务器 5900 端口

```bash
firewall-cmd --add-port=5900/tcp
```

使用 VNC Viewer 服务器的 5900 端口进行虚拟机系统安装。

## 虚拟机管理
