# OpenStack Rocky 集群部署

## 环境

### controller1

*   操作系统

    CentOS 7.6.1810
*   网络

    ens3: 192.168.2.171

    ens4: 未配置
*   硬盘

    /dev/vda: 30G 系统盘

    /dev/vdb: 30G 空闲盘

### controller2

*   操作系统

    CentOS 7.6.1810
*   网络

    ens3: 192.168.2.172

    ens4: 未配置
*   硬盘

    /dev/vda: 30G 系统盘

    /dev/vdb: 30G 空闲盘

### controller3

*   操作系统

    CentOS 7.6.1810
*   网络

    ens3: 192.168.2.173

    ens4: 未配置
*   硬盘

    /dev/vda: 30G 系统盘

    /dev/vdb: 30G 空闲盘

## 修改 SELINUX 模式

\[all]

```bash
sed -i 's/SELINUX *= *enforcing/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
getenforce
```

## 系统优化

\[all]

```bash
# 设置进程打开文件的最大数量
sed -i '/^\* *soft *nofile.*/d' /etc/security/limits.conf
echo '* soft nofile  655350' >> /etc/security/limits.conf
egrep '^\* *soft *nofile.*' /etc/security/limits.conf

sed -i '/^\* *hard *nofile.*/d' /etc/security/limits.conf
echo '* hard nofile  655350' >> /etc/security/limits.conf
egrep '^\* *hard *nofile.*' /etc/security/limits.conf
# 设置最大进程数
sed -i '/^\* *soft *nproc.*/d' /etc/security/limits.d/20-nproc.conf
echo '* soft nproc 655350' >> /etc/security/limits.d/20-nproc.conf
egrep '^\* *soft *nproc.*' /etc/security/limits.d/20-nproc.conf

sed -i '/^root *soft *nproc.*/d' /etc/security/limits.d/20-nproc.conf
echo 'root soft nproc unlimited' >> /etc/security/limits.d/20-nproc.conf
egrep '^root *soft *nproc.*' /etc/security/limits.d/20-nproc.conf

# 配置systemctl最大打开文件和最大进程
sed -i 's/\(^\|\#\)DefaultLimitNOFILE *=.*/DefaultLimitNOFILE=655350/g' /etc/systemd/system.conf
egrep '^DefaultLimitNOFILE *=.*' /etc/systemd/system.conf

sed -i 's/\(^\|\#\)DefaultLimitNPROC *=.*/DefaultLimitNPROC=655350/g' /etc/systemd/system.conf
egrep '^DefaultLimitNPROC *=.*' /etc/systemd/system.conf

systemctl daemon-reload
# 开启IP转发功能
sed -i '/^net\.ipv4\.ip_forward *=.*/d' /etc/sysctl.conf
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
egrep '^net\.ipv4\.ip_forward *=.*' /etc/sysctl.conf
sysctl -p
```

## 配置 IP

\[controller1]

```bash
nmcli con add type ethernet ifname ens3 con-name ens3 ipv4.method manual ip4 192.168.2.171/24 gw4 192.168.2.1 ipv4.dns 114.114.114.114,8.8.8.8
```

\[controller2]

```bash
nmcli con add type ethernet ifname ens3 con-name ens3 ipv4.method manual ip4 192.168.2.172/24 gw4 192.168.2.1 ipv4.dns 114.114.114.114,8.8.8.8
```

\[controller3]

```bash
nmcli con add type ethernet ifname ens3 con-name ens3 ipv4.method manual ip4 192.168.2.173/24 gw4 192.168.2.1 ipv4.dns 114.114.114.114,8.8.8.8
```

## 配置主机名

\[controller1]

```bash
hostnamectl set-hostname controller1
```

\[controller2]

```bash
hostnamectl set-hostname controller2
```

\[controller3]

```bash
hostnamectl set-hostname controller3
```

## 配置 hosts 文件

\[all]

```bash
cat << EOF >> /etc/hosts
192.168.2.170        controller
192.168.2.171        controller1
192.168.2.172        controller2
192.168.2.173        controller3
EOF

cat /etc/hosts
ping -c 4 controller1
ping -c 4 controller2
ping -c 4 controller3
```

## 配置本地源

\[all]

```bash
mkdir /etc/yum.repos.d/bak
mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak/

echo '
[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

' > /etc/yum.repos.d/Local.repo

sed -i '/mirror.centos.org/d' /etc/hosts
echo '10.0.100.189    mirror.centos.org' >> /etc/hosts

yum clean all
yum makecache
```

## 防火墙放行

\[all]

```bash
firewall-cmd --permanent --zone=trusted --add-source=192.168.2.170 --add-source=192.168.2.171 --add-source=192.168.2.172 --add-source=192.168.2.173
firewall-cmd --reload
```

## 配置 controller1 免密登录其他节点

*   生存密钥并发送公钥到其它节点

    \[controller1]

    ```bash
    ssh-keygen
    ssh-copy-id root@controller1
    scp -r ~/.ssh root@controller2:~/
    scp -r ~/.ssh root@controller3:~/
    ```
*   验证

    \[controller1]

    ```bash
    ssh root@controller1 hostname
    ssh root@controller2 hostname
    ssh root@controller3 hostname
    ```

## 安装基础软件包

\[all]

```bash
yum install -y centos-release-openstack-rocky && \
yum install -y python-openstackclient openstack-utils && \
yum install -y openstack-selinux && \
yum install -y bash-completion wget && \
yum -y upgrade && \
reboot
```

## 配置密码及相关变量

\[all]

```bash
# 配置密码信息
mkdir -p ~/openstack
cat << EOF > ~/openstack/passwordrc
# 配置管理网络IP变量
export MY_IP=$(ping -c 1 `hostname` | egrep -o '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | awk 'NR==1 {print}')
# pacemaker 认证使用的账户密码
export HACLUSTER_PASS=HACLUSTER_PASS
# HAProxy 管理界面admin的密码
export HAPROXY_PASS=123456
# RabbitMQ openstack用户的密码
export RABBIT_PASS=RABBIT_PASS
# Keystone数据使用的密码
export KEYSTONE_DBPASS=KEYSTONE_DBPASS
# openstack admin用户密码
export ADMIN_PASS=ADMIN_PASS
# ceph集群用户 cephuser的密码
export CEPHUSER_PASS=CEPHUSER_PASS
# glance 数据库密码
export GLANCE_DBPASS=GLANCE_DBPASS
# openstack glance用户密码
export GLANCE_PASS=GLANCE_PASS
# nova nova_api nova_cell0 数据库密码
export NOVA_DBPASS=NOVA_DBPASS
# placement 数据库密码
export PLACEMENT_DBPASS=PLACEMENT_DBPASS
# openstack nova用户密码
export NOVA_PASS=NOVA_PASS
# openstack placement用户密码
export PLACEMENT_PASS=PLACEMENT_PASS
# neutron 数据库密码
export NEUTRON_DBPASS=NEUTRON_DBPASS
# openstack neutron用户密码
export NEUTRON_PASS=NEUTRON_PASS
# 元数据服务密钥
export METADATA_SECRET=METADATA_SECRET
# cinder 数据库密码
export CINDER_DBPASS=CINDER_DBPASS
# openstack cinder用户密码
export CINDER_PASS=CINDER_PASS
# openstack octavia用户密码
export OCTAVIA_PASS=OCTAVIA_PASS
# octavia 数据库密码
export OCTAVIA_DBPASS=OCTAVIA_DBPASS
EOF
. ~/openstack/passwordrc
```

## 配置网络时间

*   安装软件

    \[all]

    ```bash
    yum install -y chrony
    ```
*   设置时区

    \[all]

    ```bash
    timedatectl set-timezone "Asia/Shanghai"
    ```
*   启动服务并设置开机自启

    \[all]

    ```bash
    systemctl restart chronyd.service
    systemctl enable chronyd.service
    systemctl status chronyd.service
    ```

## Pacemaker

*   安装软件包

    \[all]

    ```bash
    yum install -y pacemaker pcs resource-agents
    ```
*   启动服务并设置开机自启

    \[all]

    ```bash
    systemctl restart pcsd.service
    systemctl enable pcsd.service
    systemctl status pcsd.service
    ```
*   设置`hacluster`用户密码

    \[all]

    ```bash
    echo $HACLUSTER_PASS | passwd --stdin hacluster
    ```
*   相互认证

    \[controller1]

    ```bash
    pcs cluster auth controller1 controller2 controller3 -u hacluster -p $HACLUSTER_PASS --force
    ```
*   创建`openstack-cluster`集群

    \[controller1]

    ```bash
    pcs cluster setup --force --name openstack-cluster controller1 controller2 controller3
    ```
*   启动集群并设置自启

    \[controller1]

    ```bash
    pcs cluster start --all
    sleep 5
    pcs cluster enable --all
    ```
*   设置集群选项

    \[controller1]

    ```bash
    pcs property set stonith-enabled=false
    pcs property set no-quorum-policy=ignore
    ```
*   创建虚拟 IP

    ```bash
    pcs resource create controller-vip IPaddr2 ip=192.168.2.170 cidr_netmask=24 nic=ens3 op monitor interval=3s
    ```
*   查看集群状态

    \[all]

    ```bash
    pcs status
    ```

## Haproxy

*   安装软件包

    \[all]

    ```bash
    yum install -y haproxy
    ```
*   修改配置文件

    \[all]

    ```bash
    cat > /etc/haproxy/haproxy.cfg << EOF
    global
      chroot  /var/lib/haproxy
      daemon
      group  haproxy
      maxconn  4000
      pidfile  /var/run/haproxy.pid
      user  haproxy

    defaults
      log  global
      maxconn  4000
      option  redispatch
      retries  3
      timeout  http-request 10s
      timeout  queue 1m
      timeout  connect 10s
      timeout  client 1m
      timeout  server 1m
      timeout  check 10s

    listen admin_stats
      stats   enable
      bind    *:8888
      mode    http
      option  httplog
      log     global
      maxconn 10
      stats   refresh 30s
      stats   uri /admin
      stats   realm haproxy
      stats   auth admin:$HAPROXY_PASS
      stats   hide-version
      stats   admin if TRUE

    EOF
    ```
*   配置内核参数以允许非本地 IP 绑定

    \[all]

    ```bash
    sed -i '/^net\.ipv4\.ip_nonlocal_bind *=.*/d' /etc/sysctl.conf
    echo 'net.ipv4.ip_nonlocal_bind = 1' >> /etc/sysctl.conf
    egrep '^net\.ipv4\.ip_nonlocal_bind *=.*' /etc/sysctl.conf
    sysctl -p
    ```
*   启动服务，并设置开机自启

    \[all]

    ```bash
    systemctl restart haproxy.service
    systemctl enable haproxy.service
    systemctl status haproxy.service
    ```
*   防火墙放行

    \[all]

    ```bash
    firewall-cmd --permanent --add-port=8888/tcp
    firewall-cmd --reload
    ```
*   验证

    浏览器访问: http://192.168.2.170:8888/admin 用户名：admin 密码：`$HAPROXY_PASS`变量

## MariaDB Galera Cluster

*   安装软件包

    \[all]

    ```bash
    yum install -y mariadb mariadb-galera-server mariadb-galera-common galera rsync
    ```
*   修改配置文件

    \[all]

    ```bash
    cat > /etc/my.cnf.d/openstack.cnf << EOF
    [mysqld]
    bind-address = `hostname`

    default-storage-engine = innodb
    innodb_file_per_table
    max_connections = 4096
    collation-server = utf8_general_ci
    character-set-server = utf8

    wsrep_provider = /usr/lib64/galera/libgalera_smm.so
    wsrep_cluster_address = "gcomm://controller1,controller2,controller3"
    wsrep_node_name = `hostname`
    wsrep_node_address = `hostname`

    EOF
    ```
*   启动集群

    \[controller1]

    ```bash
    galera_new_cluster
    ```

    \[other]

    ```bash
    systemctl restart mariadb.service
    ```

    > 注意：依次启动，成功后启动下一个
*   设置开机自启

    \[all]

    ```bash
    systemctl enable mariadb.service
    ```
*   创建`haproxy`数据库用户

    \[controller1]

    ```bash
    mysql -uroot -e 'CREATE USER IF NOT EXISTS "haproxy"@"%" IDENTIFIED BY "";'
    mysql -uroot -e 'FLUSH PRIVILEGES;'
    ```
*   配置`Haproxy`并重启服务

    \[all]

    ```bash
    cat >> /etc/haproxy/haproxy.cfg << EOF
    listen galera_cluster
      bind controller:3306
      timeout  client 8h
      timeout  server 8h
      balance  source
      option  mysql-check user haproxy
      server controller1 controller1:3306 check port 3306 inter 2000 rise 2 fall 5
      server controller2 controller2:3306 backup check port 3306 inter 2000 rise 2 fall 5
      server controller3 controller3:3306 backup check port 3306 inter 2000 rise 2 fall 5

    EOF
    # 重启服务
    systemctl restart haproxy.service
    ```
*   验证`haproxy`连接

    \[all]

    ```bash
    mysql -uhaproxy -h controller -e 'SHOW STATUS LIKE "wsrep%";'
    ```

## RabbitMQ Cluster

*   安装软件包

    \[all]

    ```bash
    yum install -y rabbitmq-server
    ```
*   配置监听 IP

    \[all]

    ```bash
    cat > /etc/rabbitmq/rabbitmq-env.conf << EOF
    RABBITMQ_NODE_IP_ADDRESS=$MY_IP
    EOF
    ```
*   单个节点启动，并将 cookie 负责给其他节点

    \[controller1]

    ```bash
    systemctl restart rabbitmq-server.service
    systemctl status rabbitmq-server.service
    systemctl enable rabbitmq-server.service
    scp /var/lib/rabbitmq/.erlang.cookie root@controller2:/var/lib/rabbitmq/.erlang.cookie
    scp /var/lib/rabbitmq/.erlang.cookie root@controller3:/var/lib/rabbitmq/.erlang.cookie
    ```
*   在其它节点，修改 cookie 文件所有者及权限，设置 rabbitmq 开机自启并启动

    \[other]

    ```bash
    chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
    chmod 400 /var/lib/rabbitmq/.erlang.cookie
    systemctl restart rabbitmq-server.service
    systemctl status rabbitmq-server.service
    systemctl enable rabbitmq-server.service
    ```
*   除第一个节点外，在每个节点上运行以下命令

    \[other]

    ```bash
    rabbitmqctl stop_app
    rabbitmqctl join_cluster --ram rabbit@controller1
    rabbitmqctl start_app
    ```
*   验证节点是否正在运行

    \[all]

    ```bash
    rabbitmqctl cluster_status
    ```
*   要确保在所有正在运行的节点上镜像除具有自动生成的名称的队列之外的所有队列，请`ha-mode`通过在其中一个节点上运行以下命令将策略密钥设置为 all

    \[controller1]

    ```bash
    rabbitmqctl set_policy ha-all '^(?!amq\.).*' '{"ha-mode": "all"}'
    ```
*   创建`openstack`用户并设置权限

    \[controller1]

    ```bash
    rabbitmqctl add_user openstack $RABBIT_PASS
    rabbitmqctl set_user_tags openstack administrator
    rabbitmqctl set_permissions openstack ".*" ".*" ".*"
    ```
*   验证

    \[all]

    ```bash
    rabbitmqctl authenticate_user openstack $RABBIT_PASS
    ```

## Memcached

*   安装软件包

    \[all]

    ```bash
    yum install -y memcached python-memcached
    ```
*   修改配置文件

    \[all]

    ```bash
    sed -i 's/^OPTIONS *= *\".*\"$/OPTIONS="-l 0.0.0.0"/g' /etc/sysconfig/memcached
    egrep '^OPTIONS *= *\".*\"$' /etc/sysconfig/memcached

    sed -i 's/^MAXCONN *= *\".*\"$/MAXCONN="4096"/g' /etc/sysconfig/memcached
    egrep '^MAXCONN *= *\".*\"$' /etc/sysconfig/memcached

    sed -i 's/^CACHESIZE *= *\".*\"$/CACHESIZE="1024"/g' /etc/sysconfig/memcached
    egrep '^CACHESIZE *= *\".*\"$' /etc/sysconfig/memcached
    ```
*   启动服务并设置开机自启

    \[all]

    ```bash
    systemctl restart memcached.service
    systemctl enable memcached.service
    systemctl status memcached.service
    ```

## Ceph Cluster

*   创建 ceph 用户`cephuser`,并设置免密 sudo 权限

    \[all]

    ```bash
    getent group cephuser > /dev/null || groupadd -r cephuser
    getent passwd cephuser > /dev/null || useradd -g cephuser -d /home/cephuser -m cephuser
    # echo $CEPHUSER_PASS | passwd cephuser --stdin
    echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
    sudo chmod 0440 /etc/sudoers.d/cephuser
    ```
*   设置 controller1 节点`cephuser`用户互信

    \[all]

    ```bash
    mkdir -p /home/cephuser/.ssh
    chown -R cephuser:cephuser /home/cephuser/.ssh
    chmod -R 700 /home/cephuser/.ssh
    cat ~/.ssh/authorized_keys > /home/cephuser/.ssh/authorized_keys
    chown cephuser:cephuser /home/cephuser/.ssh/authorized_keys
    chmod 600 /home/cephuser/.ssh/authorized_keys
    ```
*   验证互信

    \[controller1]

    ```bash
    ssh cephuser@controller1 echo '$USER:$HOSTNAME'
    ssh cephuser@controller2 echo '$USER:$HOSTNAME'
    ssh cephuser@controller3 echo '$USER:$HOSTNAME'
    ```
*   安装部署工具 `ceph-deploy`

    \[all]

    ```bash
    wget https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/p/python2-pip-8.1.2-8.el7.noarch.rpm
    yum install -y python2-pip-8.1.2-8.el7.noarch.rpm
    # 更换pip源
    mkdir ~/.pip
    cat > ~/.pip/pip.conf << EOF
    [global]
    index-url = https://mirrors.aliyun.com/pypi/simple/
    [install]
    trusted-host=mirrors.aliyun.com
    EOF
    # 安装ceph-deploy
    pip install ceph-deploy==1.5.39
    ```
*   创建部署临时文件目录

    \[controller1]

    ```bash
    mkdir ~/my-cluster
    cd ~/my-cluster
    ```
*   清理环境

    \[controller1]

    ```bash
    ceph-deploy --username cephuser purge controller1 controller2 controller3
    ceph-deploy --username cephuser purgedata controller1 controller2 controller3
    ceph-deploy --username cephuser forgetkeys
    ```
*   安装软件包

    \[all]

    ```bash
    yum install -y ceph ceph-common ceph-radosgw
    ```
*   创建集群

    \[controller1]

    ```bash
    ceph-deploy --username cephuser new controller1 controller2 controller3
    ```
*   设置副本数

    \[controller1]

    ```bash
    openstack-config --set ceph.conf 'global' 'osd pool default size' '2'
    ```
*   初始 monitor(s)、并收集所有密钥

    \[controller1]

    ```bash
    ceph-deploy --username cephuser mon create-initial
    ```
*   复制密钥到您的管理节点和你的`Ceph`的节点，以便您可以使用`ceph CLI`

    \[controller]

    ```bash
    ceph-deploy --username cephuser admin controller1 controller2 controller3
    ```
*   部署管理器守护程序

    \[controller1]

    ```bash
    ceph-deploy --username cephuser mgr create controller1 controller2 controller3
    ```
*   创建 OSD

    \[controller1]

    ```bash
    ceph-deploy --username cephuser disk zap controller1:/dev/vdb controller2:/dev/vdb controller3:/dev/vdb
    ceph-deploy --username cephuser osd create controller1:/dev/vdb controller2:/dev/vdb controller3:/dev/vdb
    ```
*   查看状态

    \[all]

    ```bash
    ceph -s
    cd ~
    ```

## Identity service

*   创建数据库及用户

    \[controller1]

    ```bash
    mysql -uroot -e 'CREATE DATABASE IF NOT EXISTS keystone;'
    mysql -uroot -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
    IDENTIFIED BY '$KEYSTONE_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
    IDENTIFIED BY '$KEYSTONE_DBPASS';"
    mysql -uroot -e 'FLUSH PRIVILEGES;'
    ```
*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-keystone httpd mod_wsgi
    ```
*   修改配置文件

    \[all]

    ```bash
    openstack-config --set /etc/keystone/keystone.conf 'database' 'connection' "mysql+pymysql://keystone:$KEYSTONE_DBPASS@controller/keystone"
    openstack-config --set /etc/keystone/keystone.conf 'token' 'provider' 'fernet'
    openstack-config --set /etc/keystone/keystone.conf 'token' 'expiration' '28800'  # 3600*8
    # 配置token缓存，fernet类型不需要 缓存
    # openstack-config --set /etc/keystone/keystone.conf 'cache' 'enabled' 'true'
    # openstack-config --set /etc/keystone/keystone.conf 'cache' 'backend' 'oslo_cache.memcache_pool'
    # openstack-config --set /etc/keystone/keystone.conf 'cache' 'memcache_servers' 'controller1:11211,controller2:11211,controller3:11211'
    ```
*   填充数据库

    \[controller1]

    ```bash
    su -s /bin/sh -c "keystone-manage db_sync" keystone
    ```
*   初始化 Fernet 令牌

    \[controller1]

    ```bash
    keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
    keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
    ```
*   拷贝 Fernet 令牌到其它节点

    \[controller1]

    ```bash
    scp -r /etc/keystone/fernet-keys/ /etc/keystone/credential-keys/ root@controller2:/etc/keystone/
    scp -r /etc/keystone/fernet-keys/ /etc/keystone/credential-keys/ root@controller3:/etc/keystone/
    ```
*   设置 Fernet 密钥存储库文件权限

    \[all]

    ```bash
    chown -R keystone:keystone /etc/keystone/credential-keys/
    chown -R keystone:keystone /etc/keystone/fernet-keys/
    ```
*   引导 Identity 服务

    \[controller1]

    ```bash
    keystone-manage bootstrap --bootstrap-password $ADMIN_PASS \
      --bootstrap-admin-url http://192.168.2.170:5000/v3/ \
      --bootstrap-internal-url http://192.168.2.170:5000/v3/ \
      --bootstrap-public-url http://192.168.2.170:5000/v3/ \
      --bootstrap-region-id RegionOne
    ```
*   配置`wsgi-keystone.conf`和`httpd.conf`

    \[all]

    ```bash
    sed -i "s/^Listen *5000$/Listen `hostname`:5000/g" /usr/share/keystone/wsgi-keystone.conf
    sed -i "s/^Listen *80$/Listen `hostname`:80/g" /etc/httpd/conf/httpd.conf
    egrep '^Listen *5000$' /usr/share/keystone/wsgi-keystone.conf
    egrep '^Listen *80$' /etc/httpd/conf/httpd.conf
    ```
*   创建`wsgi-keystone.conf`链接

    \[all]

    ```bash
    ln -sf /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
    ```
*   启动`httpd`并设置开机自启

    \[all]

    ```bash
    systemctl restart httpd.service
    systemctl enable httpd.service
    systemctl status httpd.service
    ```
*   修改`haproxy`配置并重启服务

    \[all]

    ```bash
    cat >> /etc/haproxy/haproxy.cfg << EOF
    listen keystone_cluster
      bind controller:5000
      balance  source
      option  tcpka
      option  httpchk
      option  tcplog
      server controller1 controller1:5000 check inter 2000 rise 2 fall 5
      server controller2 controller2:5000 check inter 2000 rise 2 fall 5
      server controller3 controller3:5000 check inter 2000 rise 2 fall 5

    EOF
    # 重启haproxy
    systemctl restart haproxy.service
    ```
*   配置管理帐户

    \[controller1]

    ```bash
    export OS_USERNAME=admin
    export OS_PASSWORD=$ADMIN_PASS
    export OS_PROJECT_NAME=admin
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_AUTH_URL=http://controller:5000/v3
    export OS_IDENTITY_API_VERSION=3
    ```
*   创建`service`项目

    \[controller1]

    ```bash
    openstack project create --domain default \
      --description "Service Project" service
    ```
*   创建`user`角色

    \[controller1]

    ```bash
    openstack role create user
    ```
*   `admin`用户设置默认项目`admin`

    \[controller1]

    ```bash
    openstack user set --project admin admin
    ```
*   取消`OS_AUTH_URL`和`OS_PASSWORD`环境变量

    \[controller1]

    ```bash
    unset OS_AUTH_URL OS_PASSWORD
    ```
*   创建`admin-openrc`脚本

    \[all]

    ```bash
    mkdir ~/openstack
    cat > ~/openstack/admin-openrc << EOF
    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_NAME=admin
    export OS_USERNAME=admin
    export OS_PASSWORD=$ADMIN_PASS
    export OS_AUTH_URL=http://controller:5000/v3
    export OS_IDENTITY_API_VERSION=3
    export OS_IMAGE_API_VERSION=2
    EOF
    ```
*   验证

    \[all]

    ```bash
    . ~/openstack/admin-openrc
    openstack token issue
    ```

## Image service

*   创建数据库及用户

    \[controller1]

    ```bash
    mysql -uroot -e 'CREATE DATABASE glance;'
    mysql -uroot -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
      IDENTIFIED BY '$GLANCE_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
      IDENTIFIED BY '$GLANCE_DBPASS';"
    mysql -uroot -e 'FLUSH PRIVILEGES;'
    ```
*   创建`glance`用户，添加到`service`项目并设置`admin`权限

    \[controller1]

    ```bash
    openstack user create --domain default --password $GLANCE_PASS glance
    openstack role add --project service --user glance admin
    ```
*   创建`glance`服务

    \[controller1]

    ```bash
    openstack service create --name glance \
      --description "OpenStack Image" image
    ```
*   创建 api 端点

    \[controller1]

    ```bash
    openstack endpoint create --region RegionOne \
      image public http://192.168.2.170:9292
    openstack endpoint create --region RegionOne \
      image internal http://192.168.2.170:9292
    openstack endpoint create --region RegionOne \
      image admin http://192.168.2.170:9292
    ```
*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-glance
    ```
*   配置`/etc/glance/glance-api.conf`文件

    \[all]

    ```bash
    # 配置数据库连接
    openstack-config --set /etc/glance/glance-api.conf 'database' 'connection' "mysql+pymysql://glance:$GLANCE_DBPASS@controller/glance"
    # 配置认证信息
    openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
    openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
    openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
    openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'auth_type' 'password'
    openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'project_domain_name' 'Default'
    openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'user_domain_name' 'Default'
    openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'project_name' 'service'
    openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'username' 'glance'
    openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'password' "$GLANCE_PASS"
    openstack-config --set /etc/glance/glance-api.conf 'paste_deploy' 'flavor' 'keystone'
    # 配置镜像存储
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'stores' 'file,http'
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'default_store' 'file'
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'filesystem_store_datadir' '/var/lib/glance/images/'
    # 配置监控地址
    openstack-config --set /etc/glance/glance-api.conf 'DEFAULT' 'bind_host' `hostname`
    ```
*   配置`/etc/glance/glance-registry.conf`

    \[all]

    ```bash
    # 配置数据库连接
    openstack-config --set /etc/glance/glance-registry.conf 'database' 'connection' "mysql+pymysql://glance:$GLANCE_DBPASS@controller/glance"
    # 配置认证信息
    openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
    openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
    openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
    openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'auth_type' 'password'
    openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'project_domain_name' 'Default'
    openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'user_domain_name' 'Default'
    openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'project_name' 'service'
    openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'username' 'glance'
    openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'password' "$GLANCE_PASS"
    openstack-config --set /etc/glance/glance-registry.conf 'paste_deploy' 'flavor' 'keystone'
    # 配置监控地址
    openstack-config --set /etc/glance/glance-registry.conf 'DEFAULT' 'bind_host' `hostname`
    ```
*   填充数据库

    \[controller1]

    ```bash
    su -s /bin/sh -c "glance-manage db_sync" glance
    ```
*   启动服务，检查状态并设置开机自启

    \[all]

    ```bash
    systemctl restart openstack-glance-api.service \
      openstack-glance-registry.service
    systemctl enable openstack-glance-api.service \
      openstack-glance-registry.service
    systemctl status openstack-glance-api.service \
      openstack-glance-registry.service
    ```
*   加载预置元数据

    \[controller1]

    ```bash
    glance-manage db_load_metadefs
    ```
*   修改`haproxy`配置并重启服务

    \[all]

    ```bash
    cat << EOF >> /etc/haproxy/haproxy.cfg
    listen glance_api_cluster
      bind controller:9292
      balance  source
      option  tcpka
      option  httpchk
      option  tcplog
      server controller1 controller1:9292 check inter 2000 rise 2 fall 5
      server controller2 controller2:9292 check inter 2000 rise 2 fall 5
      server controller3 controller3:9292 check inter 2000 rise 2 fall 5

     listen glance_registry_cluster
      bind controller:9191
      balance  source
      option  tcpka
      option  tcplog
      server controller1 controller1:9191 check inter 2000 rise 2 fall 5
      server controller2 controller2:9191 check inter 2000 rise 2 fall 5
      server controller3 controller3:9191 check inter 2000 rise 2 fall 5

    EOF

    systemctl restart haproxy.service
    ```

## Glance 对接 Ceph

*   创建`images`存储池

    \[controller1]

    ```bash
    ceph osd pool create images 64
    rbd pool init images
    ```
*   创建`glance`用户

    \[controller1]

    ```bash
    ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images'
    ```
*   将`client.glance`密钥复制到`glance api`节点

    \[controller1]

    ```bash
    ceph auth get-or-create client.glance | ssh controller1 sudo tee /etc/ceph/ceph.client.glance.keyring
    ssh controller1 sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring

    ceph auth get-or-create client.glance | ssh controller2 sudo tee /etc/ceph/ceph.client.glance.keyring
    ssh controller2 sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring

    ceph auth get-or-create client.glance | ssh controller3 sudo tee /etc/ceph/ceph.client.glance.keyring
    ssh controller3 sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring
    ```
*   配置`/etc/glance/glance-api.conf`

    \[all]

    ```bash
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'stores' 'file,http,rbd'
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'default_store' 'rbd'
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'filesystem_store_datadir' '/var/lib/glance/images/'
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'rbd_store_pool' 'images'
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'rbd_store_user' 'glance'
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'rbd_store_ceph_conf' '/etc/ceph/ceph.conf'
    openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'rbd_store_chunk_size' '8'
    openstack-config --set /etc/glance/glance-api.conf 'DEFAULT' 'show_image_direct_url' 'True'
    ```
*   重启服务器

    \[all]

    ```bash
    systemctl restart openstack-glance-api.service \
      openstack-glance-registry.service
    systemctl status openstack-glance-api.service \
      openstack-glance-registry.service
    ```
*   验证

    \[controller1]

    ```bash
    cd ~
    # 下载cirrso镜像
    wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
    # 上传镜像
    openstack image create "cirros" \
    --file cirros-0.4.0-x86_64-disk.img \
    --disk-format qcow2 --container-format bare \
    --property hw_scsi_model=virtio-scsi \
    --property hw_disk_bus=scsi \
    --property hw_qemu_guest_agent=yes \
    --property os_require_quiesce=yes \
    --public
    # 查看镜像列表
    openstack image list
    ```

## 安装配置 Compute 服务-控制节点

*   创建数据库及用户

    \[controller1]

    ```bash
    mysql -uroot -e 'CREATE DATABASE nova_api;'
    mysql -uroot -e 'CREATE DATABASE nova;'
    mysql -uroot -e 'CREATE DATABASE nova_cell0;'
    mysql -uroot -e 'CREATE DATABASE placement;'
    mysql -uroot -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
      IDENTIFIED BY '$NOVA_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
      IDENTIFIED BY '$NOVA_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
      IDENTIFIED BY '$NOVA_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
      IDENTIFIED BY '$NOVA_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
      IDENTIFIED BY '$NOVA_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
      IDENTIFIED BY '$NOVA_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' \
      IDENTIFIED BY '$PLACEMENT_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' \
      IDENTIFIED BY '$PLACEMENT_DBPASS';"
    mysql -uroot -e 'FLUSH PRIVILEGES;'
    ```
*   创建`nova`用户，添加到`service`项目并设置`admin`权限

    \[controller1]

    ```bash
    openstack user create --domain default --password $NOVA_PASS nova
    openstack role add --project service --user nova admin
    ```
*   创建`nova`服务

    \[controller1]

    ```bash
    openstack service create --name nova \
      --description "OpenStack Compute" compute
    ```
*   创建`Nova Api`端点

    \[controller1]

    ```bash
    openstack endpoint create --region RegionOne \
      compute public http://192.168.2.170:8774/v2.1
    openstack endpoint create --region RegionOne \
      compute internal http://192.168.2.170:8774/v2.1
    openstack endpoint create --region RegionOne \
      compute admin http://192.168.2.170:8774/v2.1
    ```
*   创建`placement`用户，添加到`service`项目并设置`admin`权限

    \[controller1]

    ```bash
    openstack user create --domain default --password $PLACEMENT_PASS placement
    openstack role add --project service --user placement admin
    ```
*   创建`placement`服务

    \[controller1]

    ```bash
    openstack service create --name placement \
      --description "Placement API" placement
    ```
*   创建`Placement API` 端点

    \[controller1]

    ```bash
    openstack endpoint create --region RegionOne \
      placement public http://192.168.2.170:8778
    openstack endpoint create --region RegionOne \
      placement internal http://192.168.2.170:8778
    openstack endpoint create --region RegionOne \
      placement admin http://192.168.2.170:8778
    ```
*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-nova-api openstack-nova-conductor \
      openstack-nova-console openstack-nova-novncproxy \
      openstack-nova-scheduler openstack-nova-placement-api
    ```
*   配置`/etc/nova/nova.conf`

    \[all]

    ```bash
    #启用 compute 和metadat api
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'enabled_apis' 'osapi_compute,metadata'
    # compute 和metadat api监听地址
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'osapi_compute_listen' '$my_ip'
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'metadata_listen' '$my_ip'
    # 配置数据库连接
    openstack-config --set /etc/nova/nova.conf 'api_database' 'connection' "mysql+pymysql://nova:$NOVA_DBPASS@controller/nova_api"
    openstack-config --set /etc/nova/nova.conf 'database' 'connection' "mysql+pymysql://nova:$NOVA_DBPASS@controller/nova"
    openstack-config --set /etc/nova/nova.conf 'placement_database' 'connection' "mysql+pymysql://placement:$PLACEMENT_DBPASS@controller/placement"
    # 配置RabbitMQ连接
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'transport_url' "rabbit://openstack:$RABBIT_PASS@controller1:5672,openstack:$RABBIT_PASS@controller2:5672,openstack:$RABBIT_PASS@controller3:5672"
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'rabbit_retry_interval' '1'
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'rabbit_retry_backoff' '2'
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'rabbit_max_retries' '0'
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'amqp_durable_queues' 'true'
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'rabbit_ha_queues' 'true'
    # 配置认证信息
    openstack-config --set /etc/nova/nova.conf 'api' 'auth_strategy' 'keystone'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000/v3'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_type' 'password'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_domain_name' 'Default'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'user_domain_name' 'Default'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_name' 'service'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'username' 'nova'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'password' "$NOVA_PASS"
    #启用网络服务
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'use_neutron' 'true'
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'firewall_driver' 'nova.virt.firewall.NoopFirewallDriver'
    # 配置 VNC代理
    openstack-config --set /etc/nova/nova.conf 'vnc' 'enabled' 'true'
    openstack-config --set /etc/nova/nova.conf 'vnc' 'server_listen' '0.0.0.0'
    openstack-config --set /etc/nova/nova.conf 'vnc' 'server_proxyclient_address' '$my_ip'
    openstack-config --set /etc/nova/nova.conf 'vnc' 'novncproxy_host' '$my_ip'
    # 配置缓存
    openstack-config --set /etc/nova/nova.conf 'cache' 'enabled' 'true'
    openstack-config --set /etc/nova/nova.conf 'cache' 'backend' 'oslo_cache.memcache_pool'
    openstack-config --set /etc/nova/nova.conf 'cache' 'memcache_servers' 'controller1:11211,controller2:11211,controller3:11211'
    # 配置镜像api
    openstack-config --set /etc/nova/nova.conf 'glance' 'api_servers' 'http://controller:9292'
    # 配置锁定路径
    openstack-config --set /etc/nova/nova.conf 'oslo_concurrency' 'lock_path' '/var/lib/nova/tmp'
    # 配置Placement API
    openstack-config --set /etc/nova/nova.conf 'placement' 'region_name' 'RegionOne'
    openstack-config --set /etc/nova/nova.conf 'placement' 'project_domain_name' 'Default'
    openstack-config --set /etc/nova/nova.conf 'placement' 'project_name' 'service'
    openstack-config --set /etc/nova/nova.conf 'placement' 'auth_type' 'password'
    openstack-config --set /etc/nova/nova.conf 'placement' 'user_domain_name' 'Default'
    openstack-config --set /etc/nova/nova.conf 'placement' 'auth_url' 'http://controller:5000/v3'
    openstack-config --set /etc/nova/nova.conf 'placement' 'username' 'placement'
    openstack-config --set /etc/nova/nova.conf 'placement' 'password' "$PLACEMENT_PASS"
    # 自动发现计算节点
    openstack-config --set /etc/nova/nova.conf 'scheduler' 'discover_hosts_in_cells_interval' '300'
    # 配置本机管理节点IP
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' "$MY_IP"
    ```
*   配置`/etc/httpd/conf.d/00-nova-placement-api.conf`

    \[all]

    ```bash
    cat >> /etc/httpd/conf.d/00-nova-placement-api.conf << EOF

    <Directory /usr/bin>
       <IfVersion >= 2.4>
          Require all granted
       </IfVersion>
       <IfVersion < 2.4>
          Order allow,deny
          Allow from all
       </IfVersion>
    </Directory>
    EOF
    # 配置监控地址
    sed -i "s/^Listen.*8778$/Listen `hostname`:8778/g" /etc/httpd/conf.d/00-nova-placement-api.conf
    ```
*   重启 httpd 服务

    \[all]

    ```bash
    systemctl restart httpd
    systemctl status httpd
    ```
*   填充`nova-api`和`placement`数据库

    \[controller1]

    ```bash
    su -s /bin/sh -c "nova-manage api_db sync" nova
    ```
*   注册`cell0`数据库

    \[controller1]

    ```bash
    su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
    ```
*   创建`cell1`单元格

    \[controller1]

    ```bash
    su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
    ```
*   填充`nova`数据库

    \[controller1]

    ```bash
    su -s /bin/sh -c "nova-manage db sync" nova
    ```
*   验证 nova `cell0`和`cell1`是否正确注册

    \[controller1]

    ```bash
    su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
    ```
*   启动服务，检查状态并设置开机自启

    \[all]

    ```bash
    systemctl restart openstack-nova-api.service \
      openstack-nova-consoleauth openstack-nova-scheduler.service \
      openstack-nova-conductor.service openstack-nova-novncproxy.service
    systemctl enable openstack-nova-api.service \
      openstack-nova-consoleauth openstack-nova-scheduler.service \
      openstack-nova-conductor.service openstack-nova-novncproxy.service
    systemctl status openstack-nova-api.service \
      openstack-nova-consoleauth openstack-nova-scheduler.service \
      openstack-nova-conductor.service openstack-nova-novncproxy.service
    ```
*   修改`haproxy`配置,并重启服务

    \[all]

    ```bash
    cat << EOF >> /etc/haproxy/haproxy.cfg
    listen nova_compute_api_cluster
      bind controller:8774
      balance  source
      option  tcpka
      option  httpchk
      option  tcplog
      server controller1 controller1:8774 check inter 2000 rise 2 fall 5
      server controller2 controller2:8774 check inter 2000 rise 2 fall 5
      server controller3 controller3:8774 check inter 2000 rise 2 fall 5

    listen nova_placement_api_cluster
      bind controller:8778
      balance  source
      option  tcpka
      option  tcplog
      server controller1 controller1:8778 check inter 2000 rise 2 fall 5
      server controller2 controller2:8778 check inter 2000 rise 2 fall 5
      server controller3 controller3:8778 check inter 2000 rise 2 fall 5

    listen nova_metadata_api_cluster
      bind controller:8775
      balance  source
      option  tcpka
      option  tcplog
      server controller1 controller1:8775 check inter 2000 rise 2 fall 5
      server controller2 controller2:8775 check inter 2000 rise 2 fall 5
      server controller3 controller3:8775 check inter 2000 rise 2 fall 5

    listen nova_vncproxy_cluster
      bind controller:6080
      balance  source
      option  tcpka
      option  tcplog
      server controller1 controller1:6080 check inter 2000 rise 2 fall 5
      server controller2 controller2:6080 check inter 2000 rise 2 fall 5
      server controller3 controller3:6080 check inter 2000 rise 2 fall 5

    EOF

    systemctl restart haproxy.service
    ```
*   防火墙放行

    \[all]

    ```bash
    firewall-cmd --permanent --add-port=6080/tcp
    firewall-cmd --reload
    ```

## 安装配置 Compute 服务-计算节点

*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-nova-compute
    ```
*   配置`/etc/nova/nova.conf`

    \[all]

    ```bash
    #启用 compute 和metadat api
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'enabled_apis' 'osapi_compute,metadata'
    # 配置RabbitMQ连接
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'transport_url' "rabbit://openstack:$RABBIT_PASS@controller1:5672,openstack:$RABBIT_PASS@controller2:5672,openstack:$RABBIT_PASS@controller3:5672"
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'rabbit_retry_interval' '1'
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'rabbit_retry_backoff' '2'
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'rabbit_max_retries' '0'
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'amqp_durable_queues' 'true'
    openstack-config --set /etc/nova/nova.conf 'oslo_messaging_rabbit' 'rabbit_ha_queues' 'true'
    # 配置认证信息
    openstack-config --set /etc/nova/nova.conf 'api' 'auth_strategy' 'keystone'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000/v3'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_type' 'password'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_domain_name' 'Default'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'user_domain_name' 'Default'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_name' 'service'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'username' 'nova'
    openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'password' "$NOVA_PASS"
    #启用网络服务
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'use_neutron' 'true'
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'firewall_driver' 'nova.virt.firewall.NoopFirewallDriver'
    # 配置 VNC代理
    openstack-config --set /etc/nova/nova.conf 'vnc' 'enabled' 'true'
    openstack-config --set /etc/nova/nova.conf 'vnc' 'server_listen' '0.0.0.0'
    openstack-config --set /etc/nova/nova.conf 'vnc' 'server_proxyclient_address' '$my_ip'
    openstack-config --set /etc/nova/nova.conf 'vnc' 'novncproxy_base_url' 'http://192.168.2.170:6080/vnc_auto.html'
    # 配置镜像api
    openstack-config --set /etc/nova/nova.conf 'glance' 'api_servers' 'http://controller:9292'
    # 配置锁定路径
    openstack-config --set /etc/nova/nova.conf 'oslo_concurrency' 'lock_path' '/var/lib/nova/tmp'
    # 配置Placement API
    openstack-config --set /etc/nova/nova.conf 'placement' 'region_name' 'RegionOne'
    openstack-config --set /etc/nova/nova.conf 'placement' 'project_domain_name' 'Default'
    openstack-config --set /etc/nova/nova.conf 'placement' 'project_name' 'service'
    openstack-config --set /etc/nova/nova.conf 'placement' 'auth_type' 'password'
    openstack-config --set /etc/nova/nova.conf 'placement' 'user_domain_name' 'Default'
    openstack-config --set /etc/nova/nova.conf 'placement' 'auth_url' 'http://controller:5000/v3'
    openstack-config --set /etc/nova/nova.conf 'placement' 'username' 'placement'
    openstack-config --set /etc/nova/nova.conf 'placement' 'password' "$PLACEMENT_PASS"
    # 配置硬盘孵化时间
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'block_device_creation_timeout' '60'
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'block_device_allocate_retries' '120'
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'block_device_allocate_retries_interval' '5'
    # 配置本机管理节点IP
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' "$MY_IP"
    ```
*   配置 qemu 虚拟化

    \[all]

    ```bash
    egrep -o '(vmx|svm)' /proc/cpuinfo
    openstack-config --set /etc/nova/nova.conf 'libvirt' 'virt_type' 'qemu'
    openstack-config --set /etc/nova/nova.conf 'libvirt' 'cpm_mode' 'none'
    ```
*   配置预留资源

    \[all]

    ```bash
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'reserved_host_disk_mb' '10240'
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'reserved_host_memory_mb' '4096'
    openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'reserved_host_cpus' '1'
    ```
*   启动服务，检查状态并设置开机自启

    \[all]

    ```bash
    systemctl restart libvirtd.service openstack-nova-compute.service
    systemctl status libvirtd.service openstack-nova-compute.service
    systemctl enable libvirtd.service openstack-nova-compute.service
    ```
*   将计算节点添加到`cell` 数据库

    \[controller1]

    ```bash
    openstack compute service list --service nova-compute
    su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
    ```
*   验证

    \[all]

    ```bash
    openstack compute service list
    nova-status upgrade check
    ```

## 安装配置 Neutron-控制节点

*   创建数据库及用户

    \[controller1]

    ```bash
    mysql -uroot -e 'CREATE DATABASE neutron;'
    mysql -uroot -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
      IDENTIFIED BY '$NEUTRON_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
      IDENTIFIED BY '$NEUTRON_DBPASS';"
    mysql -uroot -e 'FLUSH PRIVILEGES;'
    ```
*   创建`neutron`用户，添加到`service`项目并设置`admin`权限。

    \[controller1]

    ```bash
    openstack user create --domain default --password $NEUTRON_PASS neutron
    openstack role add --project service --user neutron admin
    ```
*   创建`neutron`项目

    \[controller1]

    ```bash
    openstack service create --name neutron \
      --description "OpenStack Networking" network
    ```
*   创建网络服务端点

    \[controller1]

    ```bash
    openstack endpoint create --region RegionOne \
      network public http://192.168.2.170:9696
    openstack endpoint create --region RegionOne \
      network internal http://192.168.2.170:9696
    openstack endpoint create --region RegionOne \
      network admin http://192.168.2.170:9696
    ```
*   配置`haproxy`并重启服务

    \[all]

    ```bash
    cat << EOF >> /etc/haproxy/haproxy.cfg
    listen neutron_api_cluster
      bind controller:9696
      balance  source
      option  tcpka
      option  httpchk
      option  tcplog
      server controller1 controller1:9696 check inter 2000 rise 2 fall 5
      server controller2 controller2:9696 check inter 2000 rise 2 fall 5
      server controller3 controller3:9696 check inter 2000 rise 2 fall 5

    EOF

    systemctl restart haproxy.service
    ```
*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-neutron openstack-neutron-ml2 \
      openstack-neutron-linuxbridge ebtables
    ```
*   配置`/etc/neutron/neutron.conf`

    \[all]

    ```bash
    # 配置数据库连接
    openstack-config --set /etc/neutron/neutron.conf 'database' 'connection' "mysql+pymysql://neutron:$NEUTRON_DBPASS@controller/neutron"
    # 启用模块化第2层（ML2）插件，路由器服务和重叠的IP地址
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'core_plugin' 'ml2'
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'service_plugins' 'router'
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'allow_overlapping_ips' 'true'
    # 配置RabbitMQ 消息队列访问
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'transport_url' "rabbit://openstack:$RABBIT_PASS@controller1:5672,openstack:$RABBIT_PASS@controller2:5672,openstack:$RABBIT_PASS@controller3:5672"
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_retry_interval' '1'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_retry_backoff' '2'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_max_retries' '0'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'amqp_durable_queues' 'true'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_ha_queues' 'true'
    # 配置身份服务访问
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'auth_strategy' 'keystone'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_type' 'password'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_domain_name' 'Default'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'user_domain_name' 'Default'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_name' 'service'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'username' 'neutron'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'password' "$NEUTRON_PASS"
    # 配置网络以通知Compute网络拓扑更改
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'notify_nova_on_port_status_changes' 'true'
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'notify_nova_on_port_data_changes' 'true'
    openstack-config --set /etc/neutron/neutron.conf 'nova' 'auth_url' 'http://controller:5000'
    openstack-config --set /etc/neutron/neutron.conf 'nova' 'auth_type' 'password'
    openstack-config --set /etc/neutron/neutron.conf 'nova' 'project_domain_name' 'default'
    openstack-config --set /etc/neutron/neutron.conf 'nova' 'user_domain_name' 'default'
    openstack-config --set /etc/neutron/neutron.conf 'nova' 'region_name' 'RegionOne'
    openstack-config --set /etc/neutron/neutron.conf 'nova' 'project_name' 'service'
    openstack-config --set /etc/neutron/neutron.conf 'nova' 'username' 'nova'
    openstack-config --set /etc/neutron/neutron.conf 'nova' 'password' "$NOVA_PASS"
    # 配置锁定路径
    openstack-config --set /etc/neutron/neutron.conf 'oslo_concurrency' 'lock_path' '/var/lib/neutron/tmp'
    #设置DHCP 冗余
    openstack-config --set  /etc/neutron/neutron.conf 'DEFAULT' 'dhcp_agents_per_network' '3'
    #设置L3 HA
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'l3_ha' 'True'
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'allow_automatic_l3agent_failover' 'True'
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'max_l3_agents_per_router' '3'
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'min_l3_agents_per_router' '2'
    # 配置本机管理节点IP
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'bind_host' "$MY_IP"
    ```
*   配置`/etc/neutron/plugins/ml2/ml2_conf.ini`

    \[all]

    ```bash
    # 启用flat，VLAN和VXLAN网络
    openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2' 'type_drivers' 'flat,vlan,vxlan'
    # 启用VXLAN自助服务网络
    openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2' 'tenant_network_types' 'vxlan'
    # 启用Linux桥和第2层填充机制
    openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2' 'mechanism_drivers' 'linuxbridge,l2population'
    # 启用端口安全性扩展驱动程序
    openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2' 'extension_drivers' 'port_security'
    # 将提供商虚拟网络配置为扁平网络
    openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2_type_flat' 'flat_networks' 'provider'
    # 为自助服务网络配置VXLAN网络标识符范围
    openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2_type_vxlan' 'vni_ranges' '1:1000'
    # 启用ipset以提高安全组规则的效率
    openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'securitygroup' 'enable_ipset' 'true'
    ```
*   配置`/etc/neutron/plugins/ml2/linuxbridge_agent.ini`

    \[all]

    ```bash
    # 将提供者虚拟网络映射到提供者物理网络接口
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'linux_bridge' 'physical_interface_mappings' 'provider:ens4'
    # 启用VXLAN重叠网络，配置处理覆盖网络的物理网络接口的IP地址，并启用第2层填充
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'enable_vxlan' 'true'
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' "$MY_IP"
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'l2_population' 'true'
    # 启用安全组并配置Linux网桥iptables防火墙驱动程序
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'enable_security_group' 'true'
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'firewall_driver' 'neutron.agent.linux.iptables_firewall.IptablesFirewallDriver'
    ```
*   配置第 3 层代理

    \[all]

    ```bash
    # 配置Linux桥接接口驱动程序和外部网桥
    openstack-config --set /etc/neutron/l3_agent.ini 'DEFAULT' 'interface_driver' 'linuxbridge'
    ```
*   配置 DHCP 代理

    \[all]

    ```bash
    # 配置Linux桥接接口驱动程序，Dnsmasq DHCP驱动程序，并启用隔离的元数据，以便提供商网络上的实例可以通过网络访问元数据
    openstack-config --set /etc/neutron/dhcp_agent.ini 'DEFAULT' 'interface_driver' 'linuxbridge'
    openstack-config --set /etc/neutron/dhcp_agent.ini 'DEFAULT' 'dhcp_driver' 'neutron.agent.linux.dhcp.Dnsmasq'
    openstack-config --set /etc/neutron/dhcp_agent.ini 'DEFAULT' 'enable_isolated_metadata' 'true'
    ```
*   配置元数据代理

    \[all]

    ```bash
    # 配置元数据主机和共享密钥
    openstack-config --set /etc/neutron/metadata_agent.ini 'DEFAULT' 'nova_metadata_host' 'controller'
    openstack-config --set /etc/neutron/metadata_agent.ini 'DEFAULT' 'metadata_proxy_shared_secret' "$METADATA_SECRET"
    ```
*   配置 Compute 服务使用 NetWorking 服务

    \[all]

    ```bash
    # 配置访问参数，启用元数据代理并配置密码
    openstack-config --set /etc/nova/nova.conf 'neutron' 'url' 'http://controller:9696'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_url' 'http://controller:5000'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_type' 'password'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'project_domain_name' 'default'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'user_domain_name' 'default'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'region_name' 'RegionOne'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'project_name' 'service'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'username' 'neutron'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'password' "$NEUTRON_PASS"
    openstack-config --set /etc/nova/nova.conf 'neutron' 'service_metadata_proxy' 'true'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'metadata_proxy_shared_secret' "$METADATA_SECRET"
    ```
*   创建初始化脚本软链

    \[all]

    ```bash
    ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
    ```
*   填充数据库

    \[controller1]

    ```bash
    su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
      --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
    ```
*   重新启动 Compute API 服务

    \[all]

    ```bash
    systemctl restart openstack-nova-api.service
    systemctl status openstack-nova-api.service
    ```
*   启动服务，检查状态并设置开机自启

    \[all]

    ```bash
    systemctl restart neutron-server.service \
      neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
      neutron-metadata-agent.service neutron-l3-agent.service
    systemctl enable neutron-server.service \
      neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
      neutron-metadata-agent.service neutron-l3-agent.service
    systemctl status neutron-server.service \
      neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
      neutron-metadata-agent.service neutron-l3-agent.service
    ```

## 安装配置 Neutron 服务-计算节点

*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-neutron-linuxbridge ebtables ipset
    ```
*   配置`/etc/neutron/neutron.conf`

    \[all]

    ```bash
    # 配置RabbitMQ 消息队列访问
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'transport_url' "rabbit://openstack:$RABBIT_PASS@controller1:5672,openstack:$RABBIT_PASS@controller2:5672,openstack:$RABBIT_PASS@controller3:5672"
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_retry_interval' '1'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_retry_backoff' '2'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_max_retries' '0'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'amqp_durable_queues' 'true'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_ha_queues' 'true'
    # 配置身份服务访问
    openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'auth_strategy' 'keystone'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_type' 'password'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_domain_name' 'Default'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'user_domain_name' 'Default'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_name' 'service'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'username' 'neutron'
    openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'password' "$NEUTRON_PASS"
    # 配置锁定路径
    openstack-config --set /etc/neutron/neutron.conf 'oslo_concurrency' 'lock_path' '/var/lib/neutron/tmp'
    ```
*   配置 Linux 桥接代理

    \[all]

    ```bash
    # 将提供者虚拟网络映射到提供者物理网络接口
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'linux_bridge' 'physical_interface_mappings' 'provider:ens4'
    # 启用VXLAN重叠网络，配置处理覆盖网络的物理网络接口的IP地址，并启用第2层填充
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'enable_vxlan' 'true'
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' "$MY_IP"
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'l2_population' 'true'
    # 启用安全组并配置Linux网桥iptables防火墙驱动程序
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'enable_security_group' 'true'
    openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'firewall_driver' 'neutron.agent.linux.iptables_firewall.IptablesFirewallDriver'
    ```
*   配置 Compute 服务以使用 Networking 服务

    \[all]

    ```bash
    openstack-config --set /etc/nova/nova.conf 'neutron' 'url' 'http://controller:9696'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_url' 'http://controller:5000'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_type' 'password'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'project_domain_name' 'default'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'user_domain_name' 'default'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'region_name' 'RegionOne'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'project_name' 'service'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'username' 'neutron'
    openstack-config --set /etc/nova/nova.conf 'neutron' 'password' "$NEUTRON_PASS"
    ```
*   重启 Compute 服务

    \[all]

    ```bash
    systemctl restart openstack-nova-compute.service
    systemctl status openstack-nova-compute.service
    ```
*   启动服务并设置开机自启

    \[all]

    ```bash
    systemctl restart neutron-linuxbridge-agent.service
    systemctl status neutron-linuxbridge-agent.service
    systemctl enable neutron-linuxbridge-agent.service
    ```
*   检验

    \[all]

    ```bash
    openstack extension list --network
    openstack network agent list
    ```
*   创建外部网络

    \[controller1]

    ```bash
    openstack network create  --share --external \
      --provider-physical-network provider \
      --provider-network-type flat provider
    ```
*   创建外部网络子网

    \[contrller1]

    ```bash
    openstack subnet create --network provider \
      --allocation-pool start=192.168.5.10,end=192.168.5.20 \
      --dns-nameserver 114.114.114.114 --dns-nameserver 8.8.8.8 \
      --gateway 192.168.5.1 \
      --subnet-range 192.168.5.0/24 provider
    ```
*   创建内部网络

    \[controller1]

    ```bash
    openstack network create selfservice
    ```
*   创建内部网络子网

    \[controller1]

    ```bash
    openstack subnet create --network selfservice \
      --dns-nameserver 114.114.114.114 --dns-nameserver 8.8.8.8 \
      --gateway 172.16.1.1 \
      --subnet-range 172.16.1.0/24 selfservice
    ```
*   创建路由

    \[controller1]

    ```bash
    openstack router create router
    ```
*   内部网络添加路由

    \[controller1]

    ```bash
    neutron router-interface-add router selfservice
    ```
*   路由设置外部网关

    \[controller1]

    ```bash
    neutron router-gateway-set router provider
    ```

## 安装配置 Cinder 服务-控制节点

*   创建数据库及用户

    \[controller1]

    ```bash
    mysql -uroot -e 'CREATE DATABASE cinder;'
    mysql -uroot -e "GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
      IDENTIFIED BY '$CINDER_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
      IDENTIFIED BY '$CINDER_DBPASS';"
    mysql -uroot -e 'FLUSH PRIVILEGES;'
    ```
*   创建`cinder`用户，添加到`service`项目并设置`admin`权限

    \[controller1]

    ```bash
    openstack user create --domain default --password $CINDER_PASS cinder
    openstack role add --project service --user cinder admin
    ```
*   创建`cinderv2`和`cinderv3`服务实体

    \[controller1]

    ```bash
    openstack service create --name cinderv2 \
      --description "OpenStack Block Storage" volumev2
    openstack service create --name cinderv3 \
      --description "OpenStack Block Storage" volumev3
    ```
*   创建`Block Storage`服务 API 端点

    \[controller1]

    ```bash
    openstack endpoint create --region RegionOne \
      volumev2 public http://192.168.2.170:8776/v2/%\(project_id\)s
    openstack endpoint create --region RegionOne \
      volumev2 internal http://192.168.2.170:8776/v2/%\(project_id\)s
    openstack endpoint create --region RegionOne \
      volumev2 admin http://192.168.2.170:8776/v2/%\(project_id\)s

    openstack endpoint create --region RegionOne \
      volumev3 public http://192.168.2.170:8776/v3/%\(project_id\)s
    openstack endpoint create --region RegionOne \
      volumev3 internal http://192.168.2.170:8776/v3/%\(project_id\)s
    openstack endpoint create --region RegionOne \
      volumev3 admin http://192.168.2.170:8776/v3/%\(project_id\)s
    ```
*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-cinder
    ```
*   配置`/etc/cinder/cinder.conf`文件

    \[all]

    ```bash
    # 配置监听地址
    openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'osapi_volume_listen' '$my_ip'
    # 配置数据库访问
    openstack-config --set /etc/cinder/cinder.conf 'database' 'connection' "mysql+pymysql://cinder:$CINDER_DBPASS@controller/cinder"
    # 显示卷路径
    openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'show_image_direct_url' 'True'
    # 配置RabbitMQ 消息队列访问
    openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'transport_url' "rabbit://openstack:$RABBIT_PASS@controller1:5672,openstack:$RABBIT_PASS@controller2:5672,openstack:$RABBIT_PASS@controller3:5672"
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_retry_interval' '1'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_retry_backoff' '2'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_max_retries' '0'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'amqp_durable_queues' 'true'
    openstack-config --set /etc/neutron/neutron.conf 'oslo_messaging_rabbit' 'rabbit_ha_queues' 'true'
    # 配置身份服务访问
    openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'auth_strategy' 'keystone'
    openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
    openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
    openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
    openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'auth_type' 'password'
    openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'project_domain_id' 'default'
    openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'user_domain_id' 'default'
    openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'project_name' 'service'
    openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'username' 'cinder'
    openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'password' "$CINDER_PASS"
    # 配置Image服务API的位置
    openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'glance_api_servers' 'http://controller:9292'
    # 配置锁定路径
    openstack-config --set /etc/cinder/cinder.conf 'oslo_concurrency' 'lock_path' '/var/lib/cinder/tmp'
    # 配置本机管理节点IP
    openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'my_ip' "$MY_IP"
    ```
*   配置`Compute`使用`Cinder`

    \[all]

    ```bash
    openstack-config --set /etc/nova/nova.conf 'cinder' 'os_region_name' 'RegionOne'
    ```
*   填充数据库

    \[controller1]

    ```bash
    su -s /bin/sh -c "cinder-manage db sync" cinder
    ```
*   重新启动`Compute API`服务

    \[all]

    ```bash
    systemctl restart openstack-nova-api.service
    ```
*   启动`Block Storage`服务并将其配置为在系统引导时启动

    \[all]

    ```bash
    systemctl restart openstack-cinder-api.service openstack-cinder-scheduler.service
    systemctl status openstack-cinder-api.service openstack-cinder-scheduler.service
    systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
    ```
*   配置`haproxy`并重启服务

    \[all]

    ```bash
    cat << EOF >> /etc/haproxy/haproxy.cfg
    listen cinder_api_cluster
      bind controller:8776
      balance  source
      option  tcpka
      option  httpchk
      option  tcplog
      server controller1 controller1:8776 check inter 2000 rise 2 fall 5
      server controller2 controller2:8776 check inter 2000 rise 2 fall 5
      server controller3 controller3:8776 check inter 2000 rise 2 fall 5

    EOF

    systemctl restart haproxy.service
    ```

## Compute 和 Cinder 对接 Ceph 存储

*   创建存储池

    \[controller1]

    ```bash
    ceph osd pool create volumes 64
    ceph osd pool create vms 64
    rbd pool init volumes
    rbd pool init vms
    ```
*   创建`cinder`用户

    \[controller1]

    ```bash
    ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd-read-only pool=images'
    ```
*   将`client.cinder`密钥复制到`cinder-volume`和`nova-compute`节点

    \[controller1]

    ```bash
    ceph auth get-or-create client.cinder | ssh controller1 sudo tee /etc/ceph/ceph.client.cinder.keyring
    ssh controller1 sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring

    ceph auth get-or-create client.cinder | ssh controller2 sudo tee /etc/ceph/ceph.client.cinder.keyring
    ssh controller2 sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring

    ceph auth get-or-create client.cinder | ssh controller3 sudo tee /etc/ceph/ceph.client.cinder.keyring
    ssh controller3 sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring
    ```
*   在`nova-compute` 的节点上创建一个密钥的临时副本

    \[all]

    ```bash
    ceph auth get-key client.cinder > ~/client.cinder.key
    ```
*   在`nova-compute`节点上把密钥加进 libvirt 、然后删除临时副本

    \[all]

    ```bash
    cat > ~/secret.xml <<EOF
    <secret ephemeral='no' private='no'>
      <uuid>457eb676-33da-42ec-9a8c-9293d545c337</uuid>
      <usage type='ceph'>
            <name>client.cinder secret</name>
      </usage>
    </secret>
    EOF

    sudo virsh secret-define --file ~/secret.xml

    sudo virsh secret-set-value --secret 457eb676-33da-42ec-9a8c-9293d545c337 --base64 $(cat ~/client.cinder.key) && rm -f ~/client.cinder.key ~/secret.xml
    ```
*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-cinder python-keystone
    ```
*   配置`/etc/cinder/cinder.conf`文件

    \[all]

    ```bash
    # 配置Ceph存储
    openstack-config --set  /etc/cinder/cinder.conf 'DEFAULT' 'enabled_backends' 'ceph'
    openstack-config --set  /etc/cinder/cinder.conf 'DEFAULT' 'glance_api_version' '2'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'volume_driver' 'cinder.volume.drivers.rbd.RBDDriver'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'rbd_pool' 'volumes'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'rbd_ceph_conf' '/etc/ceph/ceph.conf'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'rbd_flatten_volume_from_snapshot' 'false'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'rbd_max_clone_depth' '5'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'rbd_store_chunk_size' '4'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'rados_connect_timeout' '-1'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'glance_api_version' '2'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'rbd_user' 'cinder'
    openstack-config --set  /etc/cinder/cinder.conf 'ceph' 'rbd_secret_uuid' '457eb676-33da-42ec-9a8c-9293d545c337'
    # 配置管理网络IP
    openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'my_ip' "$MY_IP"
    ```
*   配置`nova-compute`节点`/etc/nova/nova.conf`

    \[all]

    ```bash
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'images_type ' 'rbd'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'images_rbd_pool ' 'vms'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'images_rbd_ceph_conf ' '/etc/ceph/ceph.conf'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'rbd_user ' 'cinder'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'rbd_secret_uuid ' '457eb676-33da-42ec-9a8c-9293d545c337'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'disk_cachemodes' '"network=writeback"'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'inject_password' 'false'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'inject_partition' '-2'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'inject_key' 'false'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'live_migration_flag' 'VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE,VIR_MIGRATE_PERSIST_DEST,VIR_MIGRATE_TUNNELLED'
    openstack-config --set  /etc/nova/nova.conf 'libvirt' 'hw_disk_discard' 'unmap'
    ```
*   启用 rbd cache 功能

    \[all]

    ```bash
    openstack-config --set /etc/ceph/ceph.conf 'client' 'rbd cache' 'true'
    openstack-config --set /etc/ceph/ceph.conf 'client' 'rbd cache writethrough until flush' 'true'
    openstack-config --set /etc/ceph/ceph.conf 'client' 'admin socket' '/var/run/ceph/$cluster-$type.$id.$pid.$cctid.asok'
    openstack-config --set /etc/ceph/ceph.conf 'client' 'log file' '/var/log/libvirt/qemu/qemu-guest-$pid.log'
    openstack-config --set /etc/ceph/ceph.conf 'client' 'rbd concurrent management ops' '20'
    openstack-config --set /etc/ceph/ceph.conf 'client.cinder' 'keyring' '/etc/ceph/ceph.client.cinder.keyring'

    systemctl restart ceph.target
    ```
*   配置热迁移

    \[all]

    ```bash
    sed -i 's/\(^\|\#\)listen_tls.\?=.*/listen_tls = 0/g' /etc/libvirt/libvirtd.conf
    sed -i 's/\(^\|\#\)listen_tcp.\?=.*/listen_tcp = 1/g' /etc/libvirt/libvirtd.conf
    sed -i 's/\(^\|\#\)tcp_port.\?=.*/tcp_port = \"16509\"/g' /etc/libvirt/libvirtd.conf
    sed -i 's/\(^\|\#\)auth_tcp.\?=.*/auth_tcp = \"none\"/g' /etc/libvirt/libvirtd.conf
    sed -i "s/\(^\|\#\)listen_addr.\?=.*/listen_addr = \"`hostname`\"/g" /etc/libvirt/libvirtd.conf

    sed -i 's/\(^\|\#\)LIBVIRTD_ARGS.\?=.*/LIBVIRTD_ARGS = \"--listen\"/g' /etc/sysconfig/libvirtd
    ```
*   重启`nova-compute`节点服务

    \[all]

    ```bash
    systemctl restart libvirtd.service openstack-nova-compute.service
    systemctl status libvirtd.service openstack-nova-compute.service
    ```
*   启动`cinder-volume`节点服务并设置开机自启

    \[all]

    ```bash
    systemctl restart openstack-cinder-volume.service
    systemctl enable openstack-cinder-volume.service
    systemctl status openstack-cinder-volume.service
    ```

## 安装配置 Dashboard

*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-dashboard
    ```
*   修改配置

    \[all]

    ```bash
    cat >> /etc/openstack-dashboard/local_settings << EOF
    OPENSTACK_HOST = "controller"
    ALLOWED_HOSTS = ['*', ]
    SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

    CACHES = {
        'default': {
             'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
             'LOCATION': [ 'controller1:11211', 'controller2:11211', 'controller3:11211' ],
        }
    }
    OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
    OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
    OPENSTACK_API_VERSIONS = {
        "identity": 3,
        "image": 2,
        "volume": 2,
    }
    OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"
    OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
    TIME_ZONE  = "Asia/Shanghai"
    EOF
    ```
*   重启`httpd`和`Memcached`

    \[all]

    ```bash
    systemctl restart httpd.service memcached.service
    systemctl status httpd.service memcached.service
    ```
*   配置`haproxy`并重启服务

    \[all]

    ```bash
    cat << EOF >> /etc/haproxy/haproxy.cfg
    listen dashboard_cluster
      bind controller:80
      balance  source
      option  tcpka
      option  httpchk
      option  tcplog
      server controller1 controller1:80 check inter 2000 rise 2 fall 5
      server controller2 controller2:80 check inter 2000 rise 2 fall 5
      server controller3 controller3:80 check inter 2000 rise 2 fall 5

    EOF

    systemctl restart haproxy.service
    ```
*   防火墙放行

    \[all]

    ```bash
    firewall-cmd --permanent --add-port=80/tcp
    firewall-cmd --reload
    ```
*   验证

    打开浏览器，访问：http://192.168.2.170/dashboard/

    域：default 用户名：admin 密码：`$ADMIN_PASS`变量值

## 服务状态检测脚本

\[all]

```bash
echo '#!/bin/bash
function _check-service()
{
  if systemctl status $1 &> /dev/null ; then
    echo -e "\033[1;32m服务状态:运行\033[0m:$1"
  else
    echo -e "\033[1;5;31m服务状态:停止\033[0m:$1"
  fi
}

echo -e "\033[34m ----- Chrony ----- \033[0m"
_check-service 'chronyd.service'

echo -e "\033[34m ----- Pacemaker ----- \033[0m"
_check-service 'pcsd.service'

echo -e "\033[34m ----- HAProxy ----- \033[0m"
_check-service 'haproxy.service'

echo -e "\033[34m ----- MariaDB ----- \033[0m"
_check-service 'mariadb.service'

echo -e "\033[34m ----- RabbitMQ ----- \033[0m"
_check-service 'rabbitmq-server.service'

echo -e "\033[34m ----- Memcached ----- \033[0m"
_check-service 'memcached.service'

echo -e "\033[34m ----- Keystone ----- \033[0m"
_check-service 'httpd.service'

echo -e "\033[34m ----- Glance ----- \033[0m"
_check-service 'openstack-glance-api.service'
_check-service 'openstack-glance-registry.service'

echo -e "\033[34m ----- Nova Controller ----- \033[0m"
_check-service 'openstack-nova-api.service'
_check-service 'openstack-nova-consoleauth.service'
_check-service 'openstack-nova-scheduler.service'
_check-service 'openstack-nova-conductor.service'
_check-service 'openstack-nova-novncproxy.service'

echo -e "\033[34m ----- Nova Compute ----- \033[0m"
_check-service 'libvirtd.service'
_check-service 'openstack-nova-compute.service'

echo -e "\033[34m ----- Neutron Controller ----- \033[0m"
_check-service 'neutron-server.service'
_check-service 'neutron-linuxbridge-agent.service'
_check-service 'neutron-dhcp-agent.service'
_check-service 'neutron-metadata-agent.service'
_check-service 'neutron-l3-agent.service'

echo -e "\033[34m ----- Neutron Compute ----- \033[0m"
_check-service 'neutron-linuxbridge-agent.service'

echo -e "\033[34m ----- Cinder Controller ----- \033[0m"
_check-service 'openstack-cinder-api.service'
_check-service 'openstack-cinder-scheduler.service'

echo -e "\033[34m ----- Cinder Volume ----- \033[0m"
_check-service 'openstack-cinder-volume.service'
' > ~/openstack/check-openstack-service.sh
chmod +x ~/openstack/check-openstack-service.sh
# 验证
~/openstack/check-openstack-service.sh
```

## 服务重启脚本

\[all]

```bash
echo '#!/bin/bash
function _restart-service()
{
  echo -e "\033[1;33m正在重启服务\033[0m:$1"
  systemctl restart $1 &> /dev/null
  if systemctl status $1 &> /dev/null ; then
    echo -e "\033[1;32m服务状态:运行\033[0m:$1"
  else
    echo -e "\033[1;5;31m服务状态:停止\033[0m:$1"
  fi
}


echo -e "\033[34m ----- HAProxy ----- \033[0m"
_restart-service 'haproxy.service'

echo -e "\033[34m ----- MariaDB ----- \033[0m"
_restart-service 'mariadb.service'

echo -e "\033[34m ----- Keystone ----- \033[0m"
_restart-service 'httpd.service'

echo -e "\033[34m ----- Glance ----- \033[0m"
_restart-service 'openstack-glance-api.service'
_restart-service 'openstack-glance-registry.service'

echo -e "\033[34m ----- Nova Controller ----- \033[0m"
_restart-service 'openstack-nova-api.service'
_restart-service 'openstack-nova-consoleauth.service'
_restart-service 'openstack-nova-scheduler.service'
_restart-service 'openstack-nova-conductor.service'
_restart-service 'openstack-nova-novncproxy.service'

echo -e "\033[34m ----- Neutron Controller ----- \033[0m"
_restart-service 'neutron-server.service'
_restart-service 'neutron-linuxbridge-agent.service'
_restart-service 'neutron-dhcp-agent.service'
_restart-service 'neutron-metadata-agent.service'
_restart-service 'neutron-l3-agent.service'

echo -e "\033[34m ----- Cinder Controller ----- \033[0m"
_restart-service 'openstack-cinder-api.service'
_restart-service 'openstack-cinder-scheduler.service'

echo -e "\033[34m ----- Nova Compute ----- \033[0m"
_restart-service 'libvirtd.service'
_restart-service 'openstack-nova-compute.service'

echo -e "\033[34m ----- Neutron Compute ----- \033[0m"
_restart-service 'neutron-linuxbridge-agent.service'

echo -e "\033[34m ----- Cinder Volume ----- \033[0m"
_restart-service 'openstack-cinder-volume.service'
' > ~/openstack/restart-openstack-service.sh
chmod +x ~/openstack/restart-openstack-service.sh
# 验证
# ~/openstack/restart-openstack-service.sh
```

## 配置 Octavia

*   创建数据库和用户

    \[controller1]

    ```bash
    mysql -uroot -e 'CREATE DATABASE octavia;'
    mysql -uroot -e "GRANT ALL PRIVILEGES ON octavia.* TO 'octavia'@'localhost' \
      IDENTIFIED BY '$OCTAVIA_DBPASS';"
    mysql -uroot -e "GRANT ALL PRIVILEGES ON octavia.* TO 'octavia'@'%' \
      IDENTIFIED BY '$OCTAVIA_DBPASS';"
    mysql -uroot -e 'FLUSH PRIVILEGES;'
    ```
*   创建`octavia`用户，添加到`service`项目并设置`admin`权限

    \[controller1]

    ```bash
    openstack user create --domain default --password $OCTAVIA_PASS octavia
    openstack role add --project service --user octavia admin
    ```
*   创建`octavia`服务

    \[controller1]

    ```bash
    openstack service create --name octavia \
      --description "OpenStack Octavia" load-balancer
    ```
*   创建`Octavia Api`端点

    \[controller1]

    ```bash
    openstack endpoint create --region RegionOne \
      octavia public "http://192.168.2.170:9876"
    openstack endpoint create --region RegionOne \
      octavia internal "http://192.168.2.170:9876"
    openstack endpoint create --region RegionOne \
      octavia admin "http://192.168.2.170:9876"
    ```
*   切换到 Octavia 用户

    \[all]

    ```bash
    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_NAME=service
    export OS_USERNAME=octavia
    export OS_PASSWORD=$OCTAVIA_PASS
    export OS_AUTH_URL=http://controller:5000/v3
    export OS_IDENTITY_API_VERSION=3
    export OS_IMAGE_API_VERSION=2
    ```
*   安装软件包

    \[all]

    ```bash
    yum install -y openstack-octavia-api.noarch openstack-octavia-common.noarch openstack-octavia-health-manager.noarch openstack-octavia-housekeeping.noarch openstack-octavia-worker.noarch python2-octaviaclient.noarch
    ```
*   创建证书

    \[controller1]

    ```bash
    rm -rf /etc/octavia/certs
    mkdir /etc/octavia/certs
    cd /etc/octavia/certs
    mkdir newcerts private
    chmod 700 private
    touch index.txt
    echo 01 > serial
    wget https://raw.githubusercontent.com/openstack/octavia/stable/rocky/etc/certificates/openssl.cnf
    # 生成RSA私钥
    openssl genrsa -passout pass:foobar -des3 -out private/cakey.pem 2048

    # 生成签名证书
    openssl req -x509 -passin pass:foobar -new -nodes -key private/cakey.pem \
            -config ../openssl.cnf \
            -subj "/C=US/ST=Denial/L=Springfield/O=Dis/CN=www.example.com" \
            -days 18250 \
            -out ca_01.pem

    # 查看证书内容
    openssl x509 -in ca_01.pem -text -noout

    # 生成服务器密钥和csr
    openssl req \
           -newkey rsa:2048 -nodes -keyout client.key \
           -subj "/C=US/ST=Denial/L=Springfield/O=Dis/CN=www.example.com" \
           -out client.csr

    # 签名请求
    openssl ca -passin pass:foobar -config ../openssl.cnf -in client.csr \
      -days 18250 -out client-.pem -batch

    echo "生成单个pem client.pem"
    cat client-.pem client.key > client.pem
    cd -
    ```
*   拷贝证书到其它节点

    \[controller1]

    ```bash
    scp -r /etc/octavia/certs/ root@controller2:/etc/octavia/
    scp -r /etc/octavia/certs/ root@controller3:/etc/octavia/
    ```
*   修改配置文件证书部分

    \[all]

    ```bash
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'client_cert' '/etc/octavia/certs/client.pem'
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'server_ca' '/etc/octavia/certs/ca_01.pem'
    openstack-config --set /etc/octavia/octavia.conf 'certificates' 'ca_certificate' '/etc/octavia/certs/ca_01.pem'
    openstack-config --set /etc/octavia/octavia.conf 'certificates' 'ca_private_key' '/etc/octavia/certs/private/cakey.pem'
    openstack-config --set /etc/octavia/octavia.conf 'certificates' 'ca_private_key_passphrase' 'foobar'
    openstack-config --set /etc/octavia/octavia.conf 'certificates' 'server_certs_key_passphrase' 'insecure-key-do-not-use-this-key'
    ```
*   创建 ssh key 并上传

    \[controller1]

    ```bash
    rm -rf /etc/octavia/.ssh/
    mkdir -m755 /etc/octavia/.ssh/
    ssh-keygen -b 2048 -t rsa -N '' -f /etc/octavia/.ssh/octavia_ssh_key
    # 上传ssh key
    openstack keypair create --public-key /etc/octavia/.ssh/octavia_ssh_key.pub octavia_ssh_key
    ```
*   拷贝 ssh key 到其它节点

    \[controller1]

    ```bash
    scp -r /etc/octavia/.ssh/ root@controller2:/etc/octavia/
    scp -r /etc/octavia/.ssh/ root@controller3:/etc/octavia/
    ```
*   修改配置文件 ssh key 部分

    \[all]

    ```bash
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_ssh_key_name' 'octavia_ssh_key'
    ```
*   创建镜像和上传(可以在其它机器制作)

    \[controller1]

    ```bash
    # 安装创建镜像软件包
    yum install -y openstack-octavia-diskimage-create
    # 下载原始镜像
    wget http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2
    # 修改source-repository-amphora-agent文件
    # /usr/share/octavia-image-elements/amphora-agent/source-repository-amphora-agent
    # 修改amphora-agent 行，如下
    amphora-agent git /opt/amphora-agent https://github.com/openstack/octavia.git stable/rocky

    # 设置环境变量
    export DIB_LOCAL_IMAGE=/root/test/CentOS-7-x86_64-GenericCloud.qcow2

    # 创建镜像
    octavia-diskimage-create.sh -i 'centos' -s 3
    # 生成镜像在当前目录，名称为amphora-x64-haproxy.qcow2

    # 上传镜像
    openstack image create amphora-x64-haproxy --public --container-format=bare --disk-format qcow2 --file amphora-x64-haproxy.qcow2 --tag amphora
    ```
*   修改镜像部分配置文件

    \[all]

    ```bash
    image_id=$(openstack image list --property name=amphora-x64-haproxy -f value -c ID)
    owner_id=$(openstack image show ${image_id} -c owner -f value)
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_image_owner_id' "$owner_id"
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_image_tag' 'amphora'
    ```
*   创建机型

    \[controller1]

    ```bash
    openstack flavor create --id auto --ram 1024 --disk 3 --vcpus 1 --private m1.amphora -f value -c id
    ```
*   修改机型部分配置

    \[all]

    ```bash
    amp_flavor_id=$(openstack flavor show m1.amphora -f value -c id)
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_flavor_id' "$amp_flavor_id"
    ```
*   创建网络和防火墙

    \[controller1]

    ```bash
    # 创建负载管理网络和子网
    openstack network create lb-mgmt-net
    openstack subnet create --subnet-range 172.16.0.0/24 --network lb-mgmt-net lb-mgmt-subnet
    # 创建负载管理网络防火墙和规则
    openstack security group create lb-mgmt-sec-grp
    openstack security group rule create --protocol icmp lb-mgmt-sec-grp
    openstack security group rule create --protocol tcp --dst-port 22 lb-mgmt-sec-grp
    openstack security group rule create --protocol tcp --dst-port 9443 lb-mgmt-sec-grp
    # 创建负载监控管理网络防火墙和规则
    openstack security group create lb-health-mgr-sec-grp
    openstack security group rule create --protocol udp --dst-port 5555 lb-health-mgr-sec-grp
    ```
*   修改网络部分配置

    \[all]

    ```bash
    # 配置 amphora 网络
    OCTAVIA_AMP_NETWORK_ID=$(openstack network show lb-mgmt-net -f value -c id)
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_boot_network_list' "$OCTAVIA_AMP_NETWORK_ID"

    # 配置 amphora 管理网络接口防火墙
    OCTAVIA_MGMT_SEC_GRP_ID=$(openstack security group show lb-mgmt-sec-grp -f value -c id)
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_secgroup_list' "$OCTAVIA_MGMT_SEC_GRP_ID"
    ```
*   创建和配置监控网络接口

    \[all]

    ```bash
    # 创建监控网络接口并获取ID
    MGMT_PORT_ID=$(openstack port create --security-group lb-health-mgr-sec-grp --device-owner Octavia:health-mgr --host=$(hostname) -c id -f value --network lb-mgmt-net octavia-health-manager-listen-port-$(hostname))
    # 获取监控网络接口IP
    MGMT_PORT_IP=$(openstack port show -f value -c fixed_ips $MGMT_PORT_ID | awk '{FS=",| "; gsub(",",""); gsub("'\''",""); for(i = 1; i <= NF; ++i) {if ($i ~ /^ip_address/) {n=index($i, "="); if (substr($i, n+1) ~ "\\.") print substr($i, n+1)}}}')
    # 配置监控服务IP和端口
    openstack-config --set /etc/octavia/octavia.conf 'health_manager' 'bind_ip' "$MGMT_PORT_IP"
    openstack-config --set /etc/octavia/octavia.conf 'health_manager' 'bind_port' '5555'
    # 配置amphora 连接监控管理地址(未配置完成)
    openstack-config --set /etc/octavia/octavia.conf 'health_manager' 'controller_ip_port_list' "$MGMT_PORT_IP:5555"
    ```
*   物理机绑定监控管理网络接口

    \[all]

    ```bash
    # 修改dhcp文件
    sudo mkdir -m755 -p /etc/dhcp/octavia
    echo 'request subnet-mask,broadcast-address,interface-mtu;
    do-forward-updates false;
    ' > /etc/dhcp/octavia/dhclient.conf

    MGMT_PORT_MAC=$(openstack port show -c mac_address -f value $MGMT_PORT_ID)
    NETID=$(openstack network show lb-mgmt-net -c id -f value)
    BRNAME=brq$(echo $NETID|cut -c 1-11)

    # 创建接口
    sudo ip link add o-hm0 type veth peer name o-bhm0
    # 添加接口到负载管理网络的桥接接口中
    sudo brctl addif $BRNAME o-bhm0
    # 开启接口
    sudo ip link set o-bhm0 up
    # 设置mac值
    sudo ip link set dev o-hm0 address $MGMT_PORT_MAC
    # 获取IP
    sudo dhclient -v o-hm0 -cf /etc/dhcp/octavia/dhclient.conf
    ```
*   修改其它配置

    ```bash
    # 设置api服务绑定的IP和端口
    openstack-config --set /etc/octavia/octavia.conf 'api_settings' 'bind_host' "$MY_IP"
    openstack-config --set /etc/octavia/octavia.conf 'api_settings' 'bind_port' '9876'

    # 配置数据库连接
    openstack-config --set /etc/octavia/octavia.conf 'database' 'connection' "mysql+pymysql://octavia:$OCTAVIA_DBPASS@controller/octavia"

    # 配置认证信息
    openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
    openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'project_domain_name' 'Default'
    openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'project_name' 'service'
    openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'user_domain_name' 'Default'
    openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'username' 'octavia'
    openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'password' "$OCTAVIA_PASS"
    openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000/v3'
    openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'auth_type' 'password'

    # 创建amphora虚拟机的对象
    openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
    openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'project_domain_name' 'Default'
    openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'project_name' 'service'
    openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'user_domain_name' 'Default'
    openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'password' "$OCTAVIA_PASS"
    openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'username' 'octavia'
    openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'auth_type' 'password'
    openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'auth_url' 'http://controller:5000/v3'

    # 设置其他必需的默认选项
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'network_driver' 'allowed_address_pairs_driver'
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'compute_driver' 'compute_nova_driver'
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amphora_driver' 'amphora_haproxy_rest_driver'
    openstack-config --set /etc/octavia/octavia.conf 'health_manager' 'heartbeat_key' 'insecure'
    openstack-config --set /etc/octavia/octavia.conf  'house_keeping' 'load_balancer_expiry_age' '3600'
    openstack-config --set /etc/octavia/octavia.conf  'house_keeping' 'amphora_expiry_age' '3600'
    openstack-config --set /etc/octavia/octavia.conf 'api_settings' 'api_handler ' 'queue_producer'
    openstack-config --set /etc/octavia/octavia.conf 'DEFAULT' 'transport_url' "rabbit://openstack:$RABBIT_PASS@controller1:5672,openstack:$RABBIT_PASS@controller2:5672,openstack:$RABBIT_PASS@controller3:5672"
    openstack-config --set /etc/octavia/octavia.conf 'oslo_messaging' 'topic' 'octavia_prov'
    openstack-config --set /etc/octavia/octavia.conf 'oslo_messaging' 'rpc_thread_pool_size' '2'
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'loadbalancer_topology' 'SINGLE'
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'base_path' '/var/lib/octavia'
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'base_cert_dir' '/var/lib/octavia/certs'
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'bind_host' '0.0.0.0'
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'bind_port' '9443'

    # 配置neutron
    openstack-config --set /etc/neutron/neutron.conf 'octavia' 'request_poll_timeout' '3000'
    openstack-config --set /etc/neutron/neutron.conf 'octavia' 'base_url' 'http://controller:9876'

    # 用于tempest运行的devstack优化
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'connection_max_retries' '1500'
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'connection_retry_interval' '10'
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'rest_request_conn_timeout' '10'
    openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'rest_request_read_timeout' '120'
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_active_retries' '10'
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_active_wait_sec' '10'
    openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'workers' '4'
    ```
*   设置文件权限

    \[all]

    ```bash
    chown -R octavia:octavia /etc/octavia/
    ```
*   填充数据库

    \[controller1]

    ```bash
    octavia-db-manage upgrade head
    ```
*   启动服务

    \[all]

    ```bash
    systemctl restart octavia-api.service
    systemctl restart octavia-worker.service
    systemctl restart octavia-housekeeping.service
    systemctl restart octavia-health-manager.service
    ```
*   配置`haproxy`并重启服务

    \[all]

    ```bash
    cat << EOF >> /etc/haproxy/haproxy.cfg
    listen octavia_api_cluster
      bind controller:9696
      balance  source
      option  tcpka
      option  httpchk
      option  tcplog
      server controller1 controller1:9876 check inter 2000 rise 2 fall 5
      server controller2 controller2:9876 check inter 2000 rise 2 fall 5
      server controller3 controller3:9876 check inter 2000 rise 2 fall 5

    EOF

    systemctl restart haproxy.service
    ```
