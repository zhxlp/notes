## ceph-deploy 部署 ceph nautilus 版本

## 环境

操作系统：CentOS7.6.1810

ceph：ceph nautilus

## 公共部分

- 配置 hosts

  ```bash
  tee -a /etc/hosts << EOF
  192.168.200.11 node1
  192.168.200.12 node2
  192.168.200.13 node3
  EOF
  ```

- 安装源

  ```bash
  yum -y install epel-release
  yum install -y https://download.ceph.com/rpm-nautilus/el7/noarch/ceph-release-1-0.el7.noarch.rpm
  ```

- 安装时间同步 chrony

  ```bash
  yum install -y chrony
  systemctl restart chronyd
  systemctl enable chronyd
  ```

- 安装 ceph-deploy

  ```bash
  yum install -y ceph-deploy
  ```

- 安装其它软件包

  ```bash
  yum install -y crudini
  ```

- 配置节点互相

  ```bash
  ssh-keygen
  ssh-copy-id node1
  ssh-copy-id node2
  ssh-copy-id node3
  ```

- 防火墙放行端口

  ```bash
  firewall-cmd --zone=public --add-service=ceph-mon
  firewall-cmd --zone=public --add-service=ceph-mon --permanent
  firewall-cmd --zone=public --add-service=ceph
  firewall-cmd --zone=public --add-service=ceph --permanent
  ```

- 关闭 SeLinux

  ```bash
  setenforce 0
  ```

## 创建集群

- 创建临时工作目录

  ```bash
  mkdir my-cluster
  cd my-cluster
  ```

- 清理环境

  当安装失败并且想重新开始，请执行一下命令清理 ceph 软件包，数据及配置

  ```bash
  # yum -y -q remove ceph ceph-release ceph-common ceph-radosgw
  ceph-deploy purge node1 node2 node3
  # rm -rf --one-file-system -- /var/lib/ceph
  # rm -rf --one-file-system -- /etc/ceph
  ceph-deploy purgedata node1 node2 node3
  # rm -rf *.keyring
  ceph-deploy forgetkeys
  rm -rf ceph.*
  ```

- 创建集群

  ```bash
  ceph-deploy new --cluster-network 192.168.200.0/24 --public-network 192.168.200.0/24 node1
  ```

- 修改配置文件

  ```bash
  # 设置osd默认副本数
  crudini --set ceph.conf 'global' 'osd_pool_default_size' '1'
  ```

- 安装软件包

  ```bash
  # ceph-deploy install node1 node2 node3
  yum -y install epel-release
  yum install -y https://download.ceph.com/rpm-nautilus/el7/noarch/ceph-release-1-0.el7.noarch.rpm
  yum install -y ceph ceph-radosgw
  ```

- 部署初始监视器并收集密钥

  ```bash
  ceph-deploy mon create-initial
  ```

- 复制客户端管理密钥，以便使用 ceph cli

  ```bash
  ceph-deploy admin node1
  ```

- 添加 MGR

  ```bash
  ceph-deploy mgr create node1
  ```

- 添加 OSD

  ```bash
  ceph-deploy osd create --data /dev/sdb node1
  ```

- 添加 MON

  ```bash
  ceph-deploy mon add node2
  # 修改ceph.conf
  # 修改 mon_initial_members,mon_host
  crudini --set ceph.conf 'global' 'mon_initial_members' 'node1,node2'
  crudini --set ceph.conf 'global' 'mon_host' '192.168.200.11,192.168.200.12'
  # 推送配置文件
  ceph-deploy --overwrite-conf config push node1 node2
  ```

- 添加 MDS

  ```bash
  ceph-deploy mds create node1
  ```

- 添加 RGW

  ```bash
  ceph-deploy rgw create node1
  ```
