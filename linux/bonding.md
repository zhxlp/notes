# Bonding

使用 nmcli 工具实现 Linux 多网卡绑定。本文参考于https://www.jianshu.com/p/4581f81bd41f

## 环境

操作系统：CentOS7.3

网卡：ens38 ,ens39

网段：192.168.200.0/24

网关：192.168.200.2

ens38：192.168.200.128/24

ens39：192.168.200.129/24

## 操作

创建`bond0`虚拟网卡。

```bash
nmcli connection add type bond con-name bond0 ifname bond0 mode balance-alb ipv4.method manual ipv4.addresses 192.168.200.10/24 ipv4.gateway 192.168.200.2 ipv4.dns 114.114.114.114
```

配置`ens38`

```bash
nmcli connection add type bond-slave ifname ens38 master bond0
```

配置`ens39`

```bash
nmcli connection add type bond-slave ifname ens39 master bond0
```
