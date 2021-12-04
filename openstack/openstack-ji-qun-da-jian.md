# OpenStack 集群搭建

本文档使用 3 台主机搭建 OpenStack 集群，3 台均为控制节点。

## 配置 id 地址和主机名

controller 192.168.2.20

controller1 192.168.2.21

controller2 192.168.2.22

controller3 192.168.2.23

配置 hosts

[all]

```bash
echo '
192.168.2.20    controller
192.168.2.21    controller1
192.168.2.22    controller2
192.168.2.23    controller3
' >> /etc/hosts

cat /etc/hosts
```

设置主机名

[controller1]

```bash
hostnamectl set-hostname controller1
```

[controller2]

```bash
hostnamectl set-hostname controller2
```

[controller3]

```bash
hostnamectl set-hostname controller3
```

## 配置本地源

[all]

```bash
mkdir /etc/yum.repos.d/bak
mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak/

echo '
[base]
name=CentOS-$releasever - Base
baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=0
enabled=1

[updates]
name=CentOS-$releasever - Updates
baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=0
enabled=1

[extras]
name=CentOS-$releasever - Extras
baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=0
enabled=1
' > /etc/yum.repos.d/Local.repo

sed -i '/mirror.centos.org/d' /etc/hosts
echo '10.0.100.191    mirror.centos.org' >> /etc/hosts

yum clean all
```

## 防火墙放行

个节点主机相互放行

[all]

```bash
firewall-cmd --permanent --zone=trusted --add-source=192.168.2.20 --add-source=192.168.2.21 --add-source=192.168.2.22 --add-source=192.168.2.23
firewall-cmd --reload
```

## 配置 controller1 免密登录其他节点

[controller1]

```bash
ssh-keygen && \
ssh-copy-id root@controller1 ; \
ssh-copy-id root@controller2 ; \
ssh-copy-id root@controller3
```

## SELINUX

更改 SELinux 运行模式

[all]

```bash
sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/sysconfig/selinux
setenforce 0
getenforce
```

## OpenStack packages

启用 OpenStack 源,安装 Openstack Clinet，Openstack Utils，openstack-selinux，更新软件包并重启服务器。

[all]

```bash
yum install -y centos-release-openstack-newton && \
yum install -y openstack-utils && \
yum install -y python-openstackclient && \
yum install -y openstack-selinux && \
yum -y upgrade && \
reboot
```

## Network Time Protocol (NTP)

设置时区

[all]

```bash
timedatectl set-timezone "Asia/Shanghai"
```

在个节点安装`chrony`设置开机自启并启动。

[all]

```bash
yum install -y chrony && \
systemctl restart chronyd.service && \
systemctl enable chronyd.service
```

## Pacemaker

### 安装软件包

安装软件包，设置开机自启并启动。

[all]

```bash
yum install -y pacemaker pcs resource-agents && \
systemctl restart pcsd.service && \
systemctl enable pcsd.service
```

### 创建集群

设置**pcs**所需的身份验证

[all]

```bash
echo hacluster | passwd --stdin hacluster
```

[controller1]

```bash
pcs cluster auth controller1 controller2 controller3 -u hacluster -p hacluster --force
```

创建集群

[controller1]

```bash
pcs cluster setup --force --name openstack controller1 controller2 controller3
```

### 设置自启并启动集群

[controller1]

```bash
pcs cluster start --all
sleep 5
pcs cluster enable --all
```

### 设置群集选项

[controller1]

```bash
pcs property set stonith-enabled=false
pcs property set no-quorum-policy=ignore
```

### 创建 vip

[controller1]

```bash
pcs resource create controller-vip IPaddr2 ip=192.168.2.20 cidr_netmask=24 nic=ens3 op monitor interval=3s
```

### 查看集群状态

[controller1]

```bash
pcs status
```

## Haproxy

安装软件包

[all]

```bash
yum install -y haproxy
```

修复配置文件

[all]

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
  stats   auth admin:123456
  stats   hide-version
  stats   admin if TRUE

EOF
```

配置内核参数以允许非本地 IP 绑定

[all]

```bash
echo 'net.ipv4.ip_nonlocal_bind = 1' >> /etc/sysctl.conf
sysctl -p
```

将 HAProxy 添加到群集并确保 VIP 只能在 HAProxy 处于活动状态的计算机上运行

[controller1]

```bash
pcs resource create haproxy systemd:haproxy --clone
pcs constraint order start controller-vip then haproxy-clone kind=Optional
pcs constraint colocation add haproxy-clone with controller-vip
```

防火墙放行

[all]

```bash
firewall-cmd --permanent --add-port=8888/tcp
firewall-cmd --reload
```

验证

浏览器访问: http://192.168.2.20:8888/admin
用户名：admin
密码：123456

## MariaDB Galera Cluster

### 安装软件包

[all]

```bash
yum install -y mariadb mariadb-galera-server mariadb-galera-common galera rsync
```

### 配置数据库

在`controller1`节点对数据进行安全配置。

[controller1]

```bash
systemctl restart mariadb.service && \
mysql_secure_installation && \
systemctl stop mariadb.service
```

修改配置文件

[controller1]

```bash
cat > /etc/my.cnf.d/openstack.cnf << EOF
[mysqld]
bind-address = 192.168.2.21

default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8

wsrep_provider = /usr/lib64/galera/libgalera_smm.so
wsrep_cluster_address = "gcomm://controller1,controller2,controller3"
wsrep_node_name = controller1
wsrep_node_address = 192.168.2.21

EOF
```

[controller2]

```bash
cat > /etc/my.cnf.d/openstack.cnf << EOF
[mysqld]
bind-address = 192.168.2.22

default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8

wsrep_provider = /usr/lib64/galera/libgalera_smm.so
wsrep_cluster_address = "gcomm://controller1,controller2,controller3"
wsrep_node_name = controller2
wsrep_node_address = 192.168.2.22

EOF
```

[controller3]

```bash
cat > /etc/my.cnf.d/openstack.cnf << EOF
[mysqld]
bind-address = 192.168.2.23

default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8

wsrep_provider = /usr/lib64/galera/libgalera_smm.so
wsrep_cluster_address = "gcomm://controller1,controller2,controller3"
wsrep_node_name = controller3
wsrep_node_address = 192.168.2.23

EOF
```

### 启动集群

[controller1]

```bash
galera_new_cluster
```

[other]

```bash
systemctl restart mariadb.service
```

[all]

```bash
systemctl enable mariadb.service
```

### 验证集群

连接数据库，查询集信息

```bash
mysql -uroot -p -e 'SHOW STATUS LIKE "wsrep%";'
```

### 配置 Haproxy

创建`haproxy`用户。

[controller1]

```bash
mysql -uroot -p -e 'CREATE USER "haproxy"@"%" IDENTIFIED WITH "";'
```

修改 haproxy 配置文件

[all]

```bash
cat >> /etc/haproxy/haproxy.cfg << EOF
listen galera_cluster
  bind 192.168.2.20:3306
  balance  source
  option  mysql-check user haproxy
  server controller1 192.168.2.21:3306 check port 3306 inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:3306 backup check port 3306 inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:3306 backup check port 3306 inter 2000 rise 2 fall 5

EOF
```

重启`haproxy`

[controller1]

```bash
pcs resource restart haproxy
```

### 验证 Haproxy

连接 vip 地址数据库

[all]

```bash
mysql -uhaproxy -h 192.168.2.20 -e 'show databases;'
```

## RabbitMQ Cluster

安装软件包

[all]

```bash
yum install -y rabbitmq-server
```

配置监听 IP

[controller1]

```bash
cat > /etc/rabbitmq/rabbitmq-env.conf << EOF
RABBITMQ_NODE_IP_ADDRESS=192.168.2.21
EOF
```

[controller2]

```bash
cat > /etc/rabbitmq/rabbitmq-env.conf << EOF
RABBITMQ_NODE_IP_ADDRESS=192.168.2.22
EOF
```

[controller3]

```bash
cat > /etc/rabbitmq/rabbitmq-env.conf << EOF
RABBITMQ_NODE_IP_ADDRESS=192.168.2.23
EOF
```

单个节点启动，并将 cookie 负责给其他节点

[controller1]

```bash
systemctl enable rabbitmq-server.service && \
systemctl restart rabbitmq-server.service && \
scp /var/lib/rabbitmq/.erlang.cookie root@controller2:/var/lib/rabbitmq/.erlang.cookie && \
scp /var/lib/rabbitmq/.erlang.cookie root@controller3:/var/lib/rabbitmq/.erlang.cookie
```

在其它节点，修改 cookie 文件所有者及权限，设置 rabbitmq 开机自启并启动

[other]

```bash
chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie && \
chmod 400 /var/lib/rabbitmq/.erlang.cookie && \
systemctl enable rabbitmq-server.service && \
systemctl restart rabbitmq-server.service
```

除第一个节点外，在每个节点上运行以下命令

[other]

```bash
rabbitmqctl stop_app && \
rabbitmqctl join_cluster --ram rabbit@controller1 && \
rabbitmqctl start_app
```

验证节点是否正在运行

```bash
rabbitmqctl cluster_status
```

要确保在所有正在运行的节点上镜像除具有自动生成的名称的队列之外的所有队列，请`ha-mode`通过在其中一个节点上运行以下命令将策略密钥设置为 all：

[controller1]

```bash
rabbitmqctl set_policy ha-all '^(?!amq\.).*' '{"ha-mode": "all"}'
```

创建`openstack`用户并设置权限。
[controller1]

```bash
rabbitmqctl add_user openstack RABBIT_PASS
rabbitmqctl set_user_tags openstack administrator
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

验证

[all]

```bash
rabbitmqctl authenticate_user openstack RABBIT_PASS
```

## Memcached

安装软件包

[all]

```bash
yum install -y memcached python-memcached
```

修改配置文件

[controller1]

```bash
sed -i 's/^OPTIONS=\".*\"$/OPTIONS="-l 127.0.0.1,::1,192.168.2.21"/g' /etc/sysconfig/memcached
```

[controller2]

```bash
sed -i 's/^OPTIONS=\".*\"$/OPTIONS="-l 127.0.0.1,::1,192.168.2.22"/g' /etc/sysconfig/memcached
```

[controller3]

```bash
sed -i 's/^OPTIONS=\".*\"$/OPTIONS="-l 127.0.0.1,::1,192.168.2.23"/g' /etc/sysconfig/memcached
```

设置开机自启并启动

[all]

```bash
systemctl restart memcached.service
systemctl enable memcached.service
systemctl status memcached.service
```

## Identity service

### 配置数据库

在单个节点操作即可。

[controller1]

使用数据库访问客户端以`root`用户身份连接到数据库服务器 ：

```bash
mysql -uroot -p
```

创建`keystone`数据库,授予对`keystone`数据库的适当访问权限,刷新权限并退出：

```sql
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
FLUSH PRIVILEGES;
EXIT;
```

### 安装配置

安装软件包

[ALL]

```bash
yum install -y openstack-keystone httpd mod_wsgi
```

修改配置文件

[all]

```bash
openstack-config --set /etc/keystone/keystone.conf 'database' 'connection' 'mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone'
openstack-config --set /etc/keystone/keystone.conf 'token' 'provider' 'fernet'
openstack-config --set /etc/keystone/keystone.conf 'cache' 'enabled' 'true'
openstack-config --set /etc/keystone/keystone.conf 'cache' 'backend' 'oslo_cache.memcache_pool'
openstack-config --set /etc/keystone/keystone.conf 'cache' 'memcache_servers' 'controller1:11211,controller2:11211,controller3:11211'
```

初始化数据库

[controller1]

```bash
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

初始化 Fernet 密钥存储库

[controller1]

```bash
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

拷贝 Fernet 密钥存储库

[controller1]

```bash
scp -r /etc/keystone/fernet-keys/ /etc/keystone/credential-keys/ root@192.168.2.22:/etc/keystone/
scp -r /etc/keystone/fernet-keys/ /etc/keystone/credential-keys/ root@192.168.2.23:/etc/keystone/
```

设置 Fernet 密钥存储库文件权限

[all]

```bash
chown -R keystone:keystone /etc/keystone/credential-keys/
chown -R keystone:keystone /etc/keystone/fernet-keys/
```

引导身份服务

[controller1]

```bash
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://192.168.2.20:35357/v3/ \
  --bootstrap-internal-url http://192.168.2.20:35357/v3/ \
  --bootstrap-public-url http://192.168.2.20:5000/v3/ \
  --bootstrap-region-id RegionOne
```

配置`wsgi-keystone.conf和httpd.conf`

[controller1]

```bash
sed -i 's/^Listen.*5000$/Listen 192.168.2.21:5000/g' /usr/share/keystone/wsgi-keystone.conf
sed -i 's/^Listen.*35357$/Listen 192.168.2.21:35357/g' /usr/share/keystone/wsgi-keystone.conf
sed -i 's/^Listen.*80$/Listen 192.168.2.21:80/g' /etc/httpd/conf/httpd.conf
```

[controller2]

```bash
sed -i 's/^Listen.*5000$/Listen 192.168.2.22:5000/g' /usr/share/keystone/wsgi-keystone.conf
sed -i 's/^Listen.*35357$/Listen 192.168.2.22:35357/g' /usr/share/keystone/wsgi-keystone.conf
sed -i 's/^Listen.*80$/Listen 192.168.2.22:80/g' /etc/httpd/conf/httpd.conf
```

[controller3]

```bash
sed -i 's/^Listen.*5000$/Listen 192.168.2.23:5000/g' /usr/share/keystone/wsgi-keystone.conf
sed -i 's/^Listen.*35357$/Listen 192.168.2.23:35357/g' /usr/share/keystone/wsgi-keystone.conf
sed -i 's/^Listen.*80$/Listen 192.168.2.23:80/g' /etc/httpd/conf/httpd.conf
```

创建`wsgi-keystone.conf`链接

[all]

```bash
ln -sf /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

设置开机自启并启动

[all]

```bash
systemctl restart httpd.service
systemctl enable httpd.service
systemctl status httpd.service
```

### 配置 Haproxy

修改 haproxy 配置

[ALL]

```bash
cat >> /etc/haproxy/haproxy.cfg << EOF
listen keystone_admin_cluster
  bind 192.168.2.20:35357
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 192.168.2.21:35357 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:35357 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:35357 check inter 2000 rise 2 fall 5

 listen keystone_public_internal_cluster
  bind 192.168.2.20:5000
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 192.168.2.21:5000 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:5000 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:5000 check inter 2000 rise 2 fall 5

EOF
```

重启`haproxy`

[controller1]

```bash
pcs resource restart haproxy
```

### 创建域，项目，用户和角色

[controller1]

配置管理帐户

```bash
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://192.168.2.20:35357/v3
export OS_IDENTITY_API_VERSION=3
```

创建`service` 项目

```bash
openstack project create --domain default \
  --description "Service Project" service
```

创建`demo`项目

```bash
openstack project create --domain default \
  --description "Demo Project" demo
```

创建`demo`用户

```bash
openstack user create --domain default \
  --password DEMO_PASS demo
```

创建`user`角色

```bash
openstack role create user
```

将`user`角色添加到`demo`项目和用户

```bash
openstack role add --project demo --user demo user
```

### 请禁用临时身份验证令牌机制

[all]

```bash
openstack-config --set /etc/keystone/keystone-paste.ini 'pipeline:public_api' 'pipeline' 'cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension public_service'
openstack-config --set /etc/keystone/keystone-paste.ini 'pipeline:admin_api' 'pipeline' 'cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension s3_extension admin_service'
openstack-config --set /etc/keystone/keystone-paste.ini 'pipeline:api_v3' 'pipeline' 'cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension_v3 s3_extension service_v3'
```

### 创建脚本

[all]

admin 脚本

```bash
cat > ~/admin-openrc << EOF
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://192.168.2.20:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

demo 脚本

```bash
cat > ~/demo-openrc << EOF
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://192.168.2.20:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

### 验证

[all]

```bash
. ~/demo-openrc && \
openstack token issue
. ~/admin-openrc && \
openstack token issue
```

## Ceph Cluster

创建 ceph 用户`cephuser`,并设置免密 sudo 权限

[all]

```bash
useradd -d /home/cephuser -m cephuser
echo "cephuser" | passwd cephuser --stdin
echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
sudo chmod 0440 /etc/sudoers.d/cephuser
```

拷贝密钥

[controller1]

```bash
ssh-copy-id cephuser@controller1 ; \
ssh-copy-id cephuser@controller2 ; \
ssh-copy-id cephuser@controller3
```

创建部署临时文件目录

[controller1]

```bash
mkdir ~/my-cluster
cd ~/my-cluster
```

安装部署工具 `ceph-deploy`

[controller1]

```bash
yum -y install python-pip
pip install ceph-deploy==1.5.36.0
```

清理环境

[controller1]

```bash
ceph-deploy --username cephuser purge controller1 controller2 controller3
ceph-deploy --username cephuser purgedata controller1 controller2 controller3
ceph-deploy --username cephuser forgetkeys
```

创建集群

[controller1]

```bash
ceph-deploy --username cephuser new controller1 controller2 controller3
```

安装软件包

[all]

```bash
yum install -y ceph ceph-common ceph-radosgw
```

初始 monitor(s)、并收集所有密钥

[controller1]

```bash
ceph-deploy --username cephuser mon create-initial
```

创建 OSD

[controller1]

```bash
ceph-deploy --username cephuser disk zap controller1:/dev/vdb controller2:/dev/vdb controller3:/dev/vdb
ceph-deploy --username cephuser osd create controller1:/dev/vdb controller2:/dev/vdb controller3:/dev/vdb
```

## Image service

### 配置数据库

[controller1]

连接数据库

```bash
mysql -u root -p
```

创建`glance`数据库，授予对`glance`数据库的适当访问权限，刷新权限并推出。

```sql
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY 'GLANCE_DBPASS';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY 'GLANCE_DBPASS';
FLUSH PRIVILEGES;
EXIT;
```

### 创建用户，服务实体和端点

[controller1]

加载 admin 环境变量

```bash
. ~/admin-openrc
```

创建`glance`用户

```bash
openstack user create --domain default --password GLANCE_PASS glance
```

将`admin`角色添加到`glance`用户和 `service`项目

```bash
openstack role add --project service --user glance admin
```

创建`glance`服务实体

```bash
openstack service create --name glance \
  --description "OpenStack Image" image
```

创建 Image 服务 API 端点

```bash
openstack endpoint create --region RegionOne \
  image public http://192.168.2.20:9292
openstack endpoint create --region RegionOne \
  image internal http://192.168.2.20:9292
openstack endpoint create --region RegionOne \
  image admin http://192.168.2.20:9292
```

### 安装和配置组件

安装软件包

[all]

```bash
yum install -y openstack-glance
```

配置`/etc/glance/glance-api.conf`

[all]

```bash
openstack-config --set /etc/glance/glance-api.conf 'database' 'connection' 'mysql+pymysql://glance:GLANCE_DBPASS@controller/glance'
openstack-config --set /etc/glance/glance-api.conf 'DEFAULT' 'registry_host' 'controller'
openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'auth_uri' 'http://controller:5000'
openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'auth_type' 'password'
openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'project_domain_name' 'Default'
openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'user_domain_name' 'Default'
openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'project_name' 'service'
openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'username' 'glance'
openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'password' 'GLANCE_PASS'
openstack-config --set /etc/glance/glance-api.conf 'paste_deploy' 'flavor' 'keystone'
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'stores' 'file,http'
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'default_store' 'file'
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'filesystem_store_datadir' '/var/lib/glance/images/'
```

[controller1]

```bash
openstack-config --set /etc/glance/glance-api.conf 'DEFAULT' 'bind_host' '192.168.2.21'
```

[controller2]

```bash
openstack-config --set /etc/glance/glance-api.conf 'DEFAULT' 'bind_host' '192.168.2.22'
```

[controller3]

```bash
openstack-config --set /etc/glance/glance-api.conf 'DEFAULT' 'bind_host' '192.168.2.23'
```

配置`/etc/glance/glance-registry.conf`

[ALL]

```bash
openstack-config --set /etc/glance/glance-registry.conf 'database' 'connection' 'mysql+pymysql://glance:GLANCE_DBPASS@controller/glance'
openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'auth_uri' 'http://controller:5000'
openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'auth_type' 'password'
openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'project_domain_name' 'Default'
openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'user_domain_name' 'Default'
openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'project_name' 'service'
openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'username' 'glance'
openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'password' 'GLANCE_PASS'
openstack-config --set /etc/glance/glance-registry.conf 'paste_deploy' 'flavor' 'keystone'
```

[controller1]

```bash
openstack-config --set /etc/glance/glance-registry.conf 'DEFAULT' 'bind_host' '192.168.2.21'
```

[controller2]

```bash
openstack-config --set /etc/glance/glance-registry.conf 'DEFAULT' 'bind_host' '192.168.2.22'
```

[controller3]

```bash
openstack-config --set /etc/glance/glance-registry.conf 'DEFAULT' 'bind_host' '192.168.2.23'
```

填充数据库

[controller1]

```bash
su -s /bin/sh -c "glance-manage db_sync" glance
```

设置开机自启并启动

[all]

```bash
systemctl restart openstack-glance-api.service \
  openstack-glance-registry.service
systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service
systemctl status openstack-glance-api.service \
  openstack-glance-registry.service
```

### 配置 Haproxy

修改 haproxy 配置

[all]

```bash
cat << EOF >> /etc/haproxy/haproxy.cfg
listen glance_api_cluster
  bind 192.168.2.20:9292
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 192.168.2.21:9292 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:9292 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:9292 check inter 2000 rise 2 fall 5

 listen glance_registry_cluster
  bind 192.168.2.20:9191
  balance  source
  option  tcpka
  option  tcplog
  server controller1 192.168.2.21:9191 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:9191 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:9191 check inter 2000 rise 2 fall 5

EOF
```

重启`haproxy`

[controller1]

```bash
pcs resource restart haproxy
```

### 配置 ceph 存储

创建 images 池

[controller1]

```bash
ceph osd pool create images 128
```

创建 Glance 用户

[controller1]

```bash
ceph auth get-or-create client.glance mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=images'
```

client.glance 的密钥环复制到适当的节点，并更改所有权

[controller1]

```bash
ceph auth get-or-create client.glance | ssh controller1 sudo tee /etc/ceph/ceph.client.glance.keyring
ssh controller1 sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring

ceph auth get-or-create client.glance | ssh controller2 sudo tee /etc/ceph/ceph.client.glance.keyring
ssh controller2 sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring

ceph auth get-or-create client.glance | ssh controller3 sudo tee /etc/ceph/ceph.client.glance.keyring
ssh controller3 sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring
```

配置`/etc/glance/glance-api.conf`

[all]

```bash
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'stores' 'file,http,rbd'
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'default_store' 'rbd'
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'filesystem_store_datadir' '/var/lib/glance/images/'
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'rbd_store_pool' 'images'
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'rbd_store_user' 'glance'
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'rbd_store_ceph_conf' '/etc/ceph/ceph.conf'
openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'rbd_store_chunk_size' '8'
```

重启服务

[all]

```bash
systemctl restart openstack-glance-api.service \
  openstack-glance-registry.service
systemctl status openstack-glance-api.service \
  openstack-glance-registry.service
```

### 验证

[controller1]

获取 admin 环境

```bash
. admin-openrc
```

下载 cirros 镜像

```bash
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```

上传镜像

```bash
openstack image create "cirros" \
  --file cirros-0.3.4-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```

查看镜像列表

```bash
openstack image list
```

## Compute Service

### 配置数据库

[all]

连接数据库

```bash
mysql -u root -p
```

创建`nova_api`和`nova`数据库，授予对数据库的适当访问权限，刷新权限并退出。

```sql
CREATE DATABASE nova_api;
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
FLUSH PRIVILEGES;
EXIT;
```

### 创建用户，服务实体和端点

[controller1]

获取 admin 环境变量

```bash
. admin-openrc
```

创建`nova`用户

```bash
openstack user create --domain default \
  --password NOVA_PASS nova
```

将`admin`角色添加到`nova`用户

```bash
openstack role add --project service --user nova admin
```

创建`nova`服务实体

```bash
openstack service create --name nova \
  --description "OpenStack Compute" compute
```

创建 Compute 服务 API 端点

```bash
openstack endpoint create --region RegionOne \
  compute public http://192.168.2.20:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  compute internal http://192.168.2.20:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  compute admin http://192.168.2.20:8774/v2.1/%\(tenant_id\)s
```

### 安装配置控制部分

安装软件包

[ALL]

```bash
yum install -y openstack-nova-api openstack-nova-conductor \
  openstack-nova-console openstack-nova-novncproxy \
  openstack-nova-scheduler
```

配置`/etc/nova/nova.conf`

[ALL]

```bash
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'enabled_apis' 'osapi_compute,metadata'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'osapi_compute_listen' '$my_ip'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'metadata_listen' '$my_ip'
openstack-config --set /etc/nova/nova.conf 'api_database' 'connection' 'mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api'
openstack-config --set /etc/nova/nova.conf 'database' 'connection' 'mysql+pymysql://nova:NOVA_DBPASS@controller/nova'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller1:5672,openstack:RABBIT_PASS@controller2:5672,openstack:RABBIT_PASS@controller3:5672'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'auth_strategy' 'keystone'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_uri' 'http://controller:5000'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_type' 'password'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_domain_name' 'Default'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'user_domain_name' 'Default'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_name' 'service'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'username' 'nova'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'password' 'NOVA_PASS'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'use_neutron' 'True'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'firewall_driver' 'nova.virt.firewall.NoopFirewallDriver'
openstack-config --set /etc/nova/nova.conf 'vnc' 'vncserver_listen' '$my_ip'
openstack-config --set /etc/nova/nova.conf 'vnc' 'vncserver_proxyclient_address' '$my_ip'
openstack-config --set /etc/nova/nova.conf 'vnc' 'novncproxy_host' '$my_ip'
openstack-config --set /etc/nova/nova.conf 'cache' 'enabled' 'true'
openstack-config --set /etc/nova/nova.conf 'cache' 'backend' 'oslo_cache.memcache_pool'
openstack-config --set /etc/nova/nova.conf 'cache' 'memcache_servers' 'controller1:11211,controller2:11211,controller3:11211'
openstack-config --set /etc/nova/nova.conf 'glance' 'api_servers' 'http://controller:9292'
openstack-config --set /etc/nova/nova.conf 'oslo_concurrency' 'lock_path' '/var/lib/nova/tmp'
```

[controller1]

```bash
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' '192.168.2.21'
```

[controller2]

```bash
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' '192.168.2.22'
```

[controller3]

```bash
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' '192.168.2.23'
```

填充数据库

[controller1]

```bash
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage db sync" nova
```

设置服务开机自启并启动

[all]

```bash
systemctl restart openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl enable openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl status openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
```

### 配置 Haproxy

[all]

配置 haproxy 配置文件

```bash
cat << EOF >> /etc/haproxy/haproxy.cfg
listen nova_compute_api_cluster
  bind 192.168.2.20:8774
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 192.168.2.21:8774 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:8774 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:8774 check inter 2000 rise 2 fall 5

listen nova_metadata_api_cluster
  bind 192.168.2.20:8775
  balance  source
  option  tcpka
  option  tcplog
  server controller1 192.168.2.21:8775 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:8775 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:8775 check inter 2000 rise 2 fall 5

listen nova_vncproxy_cluster
  bind 192.168.2.20:6080
  balance  source
  option  tcpka
  option  tcplog
  server controller1 192.168.2.21:6080 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:6080 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:6080 check inter 2000 rise 2 fall 5

EOF
```

重启`haproxy`

[controller1]

```bash
pcs resource restart haproxy
```

### 防火墙放行

[all]

```bash
firewall-cmd --permanent --add-port=6080/tcp
firewall-cmd --reload
```

### 安装配置计算部分

安装软件包

[all]

```bash
yum install -y openstack-nova-compute
```

配置`/etc/nova/nova.conf`

[all]

```bash
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'enabled_apis' 'osapi_compute,metadata'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller1:5672,openstack:RABBIT_PASS@controller2:5672,openstack:RABBIT_PASS@controller3:5672'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'auth_strategy' 'keystone'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_uri' 'http://controller:5000'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_type' 'password'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_domain_name' 'Default'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'user_domain_name' 'Default'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_name' 'service'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'username' 'nova'
openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'password' 'NOVA_PASS'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'use_neutron' 'True'
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'firewall_driver' 'nova.virt.firewall.NoopFirewallDriver'
openstack-config --set /etc/nova/nova.conf 'vnc' 'enabled' 'True'
openstack-config --set /etc/nova/nova.conf 'vnc' 'vncserver_listen' '$my_ip'
openstack-config --set /etc/nova/nova.conf 'vnc' 'vncserver_proxyclient_address' '$my_ip'
openstack-config --set /etc/nova/nova.conf 'vnc' 'novncproxy_base_url' 'http://192.168.2.20:6080/vnc_auto.html'
openstack-config --set /etc/nova/nova.conf 'glance' 'api_servers' 'http://controller:9292'
openstack-config --set /etc/nova/nova.conf 'oslo_concurrency' 'lock_path' '/var/lib/nova/tmp'
```

[controller1]

```bash
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' '192.168.2.21'
```

[controller2]

```bash
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' '192.168.2.22'
```

[controller3]

```bash
openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' '192.168.2.23'
```

使用 qemu 虚拟类型

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

执行上面命令，结果为 0，需要设置如下

```bash
openstack-config --set /etc/nova/nova.conf 'libvirt' 'virt_type' 'qemu'
openstack-config --set /etc/nova/nova.conf 'libvirt' 'cpu_mode' 'none'
```

设置开机自启并启动

[all]

```bash
systemctl restart libvirtd.service openstack-nova-compute.service
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl status libvirtd.service openstack-nova-compute.service
```

### 验证

[all]

获取 admin 环境变量

```bash
. ~/admin-openrc
```

获取`compute`服务列表

```bash
openstack compute service list
```

## Networking Service

### 配置数据库

[controller1]

连接数据库

```bash
mysql -u root -p
```

创建`neutron`数据库,授予对数据库的适当访问权限，刷新权限并退出。

```sql
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY 'NEUTRON_DBPASS';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
  IDENTIFIED BY 'NEUTRON_DBPASS';
FLUSH PRIVILEGES;
EXIT;
```

### 创建用户，服务实体和端点

[controller1]

获取 admin 环境变量

```bash
. admin-openrc
```

创建`neutron`用户

```bash
openstack user create --domain default \
  --password NEUTRON_PASS neutron
```

将`admin`角色添加到`neutron`用户

```bash
openstack role add --project service --user neutron admin
```

创建`neutron`服务实体

```bash
openstack service create --name neutron \
  --description "OpenStack Networking" network
```

创建 Networking 服务 API 端点

```bash
openstack endpoint create --region RegionOne \
  network public http://192.168.2.20:9696
openstack endpoint create --region RegionOne \
  network internal http://192.168.2.20:9696
openstack endpoint create --region RegionOne \
  network admin http://192.168.2.20:9696
```

### 配置 Haproxy

[all]

配置 haproxy 配置文件

```bash
cat << EOF >> /etc/haproxy/haproxy.cfg
listen neutron_api_cluster
  bind 192.168.2.20:9696
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 192.168.2.21:9696 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:9696 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:9696 check inter 2000 rise 2 fall 5

EOF
```

重启`haproxy`

[controller1]

```bash
pcs resource restart haproxy
```

## 安装配置控制部分

#### 配置私有网络类型

[all]

安装软件包

```bash
yum install -y openstack-neutron openstack-neutron-ml2 \
  openstack-neutron-linuxbridge ebtables
```

##### 配置`neutron.conf`

[all]

```bash
openstack-config --set /etc/neutron/neutron.conf 'database' 'connection' 'mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron'

# 启用模块化第2层（ML2）插件，路由器服务和重叠的IP地址
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'core_plugin' 'ml2'
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'service_plugins' 'router'
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'allow_overlapping_ips' 'True'
# 配置RabbitMQ 消息队列访问
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller1:5672,openstack:RABBIT_PASS@controller2:5672,openstack:RABBIT_PASS@controller3:5672'
# 配置身份服务访问
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'auth_strategy' 'keystone'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_uri' 'http://controller:5000'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_type' 'password'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_domain_name' 'Default'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'user_domain_name' 'Default'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_name' 'service'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'username' 'neutron'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'password' 'NEUTRON_PASS'
# 配置网络以通知Compute网络拓扑更改
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'notify_nova_on_port_status_changes' 'True'
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'notify_nova_on_port_data_changes' 'True'
openstack-config --set /etc/neutron/neutron.conf 'nova' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/neutron/neutron.conf 'nova' 'auth_type' 'password'
openstack-config --set /etc/neutron/neutron.conf 'nova' 'project_domain_name' 'Default'
openstack-config --set /etc/neutron/neutron.conf 'nova' 'user_domain_name' 'Default'
openstack-config --set /etc/neutron/neutron.conf 'nova' 'region_name' 'RegionOne'
openstack-config --set /etc/neutron/neutron.conf 'nova' 'project_name' 'service'
openstack-config --set /etc/neutron/neutron.conf 'nova' 'username' 'nova'
openstack-config --set /etc/neutron/neutron.conf 'nova' 'password' 'NOVA_PASS'
#设置DHCP 冗余
openstack-config --set  /etc/neutron/neutron.conf 'DEFAULT' 'dhcp_agents_per_network' '3'
#设置L3 HA
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'l3_ha' 'True'
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'allow_automatic_l3agent_failover' 'True'
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'max_l3_agents_per_router' '3'
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'min_l3_agents_per_router' '2'
# 配置锁定路径
openstack-config --set /etc/neutron/neutron.conf 'oslo_concurrency' 'lock_path' '/var/lib/neutron/tmp'

```

[controller1]

```bash
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'bind_host' '192.168.2.21'
```

[controller2]

```bash
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'bind_host' '192.168.2.22'
```

[controller3]

```bash
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'bind_host' '192.168.2.23'
```

##### 配置`ml2_conf.ini`

[all]

```bash
# 启用flat，VLAN和VXLAN网络
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2' 'type_drivers' 'flat,vlan,vxlan'
# 普通用户创建的网络
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2' 'tenant_network_types' 'vxlan'
# 启用Linux桥和第2层填充机制
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2' 'mechanism_drivers' 'linuxbridge,l2population'
# 启用端口安全性扩展驱动程序
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2' 'extension_drivers' 'port_security'
# 将公共虚拟网络配置为扁平网络
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2_type_flat' 'flat_networks' 'provider'
# 为私有服务网络配置VXLAN网络标识符范围
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'ml2_type_vxlan' 'vni_ranges' '1:1000'
# 启用ipset以提高安全组规则的效率
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini 'securitygroup' 'enable_ipset' 'True'
```

##### 配置 Linux 桥代理

[all]

```bash
# 配置公共网络物理网络接口
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'linux_bridge' 'physical_interface_mappings' "provider:ens4"
# 启用VXLAN重叠网络并启用第2层填充
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'enable_vxlan' 'True'
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'l2_population' 'True'
# 启用安全组并配置Linux桥接iptables防火墙驱动程序
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'enable_security_group' 'True'
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'firewall_driver' 'neutron.agent.linux.iptables_firewall.IptablesFirewallDriver'

```

[controller1]

```bash
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' '192.168.2.21'
```

[controller2]

```bash
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' '192.168.2.22'
```

[controller3]

```bash
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' '192.168.2.23'
```

##### 配置第 3 层代理

[all]

```bash
openstack-config --set /etc/neutron/l3_agent.ini 'DEFAULT' 'interface_driver' 'neutron.agent.linux.interface.BridgeInterfaceDriver'
```

##### 配置 DHCP 代理

[all]

```bash
openstack-config --set /etc/neutron/dhcp_agent.ini 'DEFAULT' 'interface_driver' 'neutron.agent.linux.interface.BridgeInterfaceDriver'
openstack-config --set /etc/neutron/dhcp_agent.ini 'DEFAULT' 'dhcp_driver' 'neutron.agent.linux.dhcp.Dnsmasq'
openstack-config --set /etc/neutron/dhcp_agent.ini 'DEFAULT' 'enable_isolated_metadata' 'True'
```

#### 配置元数据代理

[all]

```bash
openstack-config --set /etc/neutron/metadata_agent.ini 'DEFAULT' 'nova_metadata_ip' '192.168.2.20'
openstack-config --set /etc/neutron/metadata_agent.ini 'DEFAULT' 'metadata_proxy_shared_secret' 'METADATA_SECRET'
```

#### 配置 Compute 服务以使用 Networking 服务

[all]

```bash
openstack-config --set /etc/nova/nova.conf 'neutron' 'url' 'http://controller:9696'
openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_type' 'password'
openstack-config --set /etc/nova/nova.conf 'neutron' 'project_domain_name' 'Default'
openstack-config --set /etc/nova/nova.conf 'neutron' 'user_domain_name' 'Default'
openstack-config --set /etc/nova/nova.conf 'neutron' 'region_name' 'RegionOne'
openstack-config --set /etc/nova/nova.conf 'neutron' 'project_name' 'service'
openstack-config --set /etc/nova/nova.conf 'neutron' 'username' 'neutron'
openstack-config --set /etc/nova/nova.conf 'neutron' 'password' 'NEUTRON_PASS'
openstack-config --set /etc/nova/nova.conf 'neutron' 'service_metadata_proxy' 'True'
openstack-config --set /etc/nova/nova.conf 'neutron' 'metadata_proxy_shared_secret' 'METADATA_SECRET'

```

#### 完成安装

创建 ML2 插件配置文件的连接

[all]

```bash
ln -sf /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

填充数据库

[controller1]

```bash
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

重新启动 Compute API 服务

[all]

```bash
systemctl restart openstack-nova-api.service
```

设置网络服务开启自启并启动

[all]

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

### 安装配置计算部分

[all]

安装软件包

```bash
yum install -y openstack-neutron-linuxbridge ebtables ipset
```

#### 配置`neutron.conf`

```bash
# 配置RabbitMQ 消息队列访问
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller1:5672,openstack:RABBIT_PASS@controller2:5672,openstack:RABBIT_PASS@controller3:5672'
# 配置身份服务访问
openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'auth_strategy' 'keystone'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_uri' 'http://controller:5000'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_type' 'password'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_domain_name' 'Default'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'user_domain_name' 'Default'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_name' 'service'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'username' 'neutron'
openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'password' 'NEUTRON_PASS'
# 配置锁定路径
openstack-config --set /etc/neutron/neutron.conf 'oslo_concurrency' 'lock_path' '/var/lib/neutron/tmp'
```

#### 配置私有网络类型

##### 配置 Linux 桥代理

[all]

```bash
# 配置公共网络物理网络接口
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'linux_bridge' 'physical_interface_mappings' "provider:ens4"
# 启用VXLAN重叠网络并启用第2层填充
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'enable_vxlan' 'True'
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'l2_population' 'True'
# 启用安全组并配置Linux桥接iptables防火墙驱动程序
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'enable_security_group' 'True'
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'firewall_driver' 'neutron.agent.linux.iptables_firewall.IptablesFirewallDriver'

```

[controller1]

```bash
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' '192.168.2.21'
```

[controller2]

```bash
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' '192.168.2.22'
```

[controller3]

```bash
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' '192.168.2.23'
```

#### 配置 Compute 服务以使用 Networking 服务

[all]

```bash
openstack-config --set /etc/nova/nova.conf 'neutron' 'url' 'http://controller:9696'
openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_type' 'password'
openstack-config --set /etc/nova/nova.conf 'neutron' 'project_domain_name' 'Default'
openstack-config --set /etc/nova/nova.conf 'neutron' 'user_domain_name' 'Default'
openstack-config --set /etc/nova/nova.conf 'neutron' 'region_name' 'RegionOne'
openstack-config --set /etc/nova/nova.conf 'neutron' 'project_name' 'service'
openstack-config --set /etc/nova/nova.conf 'neutron' 'username' 'neutron'
openstack-config --set /etc/nova/nova.conf 'neutron' 'password' 'NEUTRON_PASS'
```

#### 完成安装

[all]

重启 Compute 服务

```bash
systemctl restart openstack-nova-compute.service
```

设置 Linux 网桥代理开机自启并启动

```bash
systemctl restart neutron-linuxbridge-agent.service
systemctl enable neutron-linuxbridge-agent.service
systemctl status neutron-linuxbridge-agent.service
```

### 验证

[controller1]

获取 admin 环境变量

```bash
. admin-openrc
```

获取`neutron`代理列表

```bash
openstack network agent list
```

### 创建网络

[controller1]

创建外部网络

```bash
openstack network create  --share --external \
  --provider-physical-network provider \
  --provider-network-type flat provider
```

创建外部网络子网

```bash
openstack subnet create --network provider \
  --allocation-pool start=192.168.5.200,end=192.168.5.220 \
  --dns-nameserver 114.114.114.114 --gateway 192.168.5.1 \
  --subnet-range 192.168.5.0/24 provider
```

创建内部网络

```bash
openstack network create selfservice
```

创建内部网络子网

```bash
openstack subnet create --network selfservice \
  --dns-nameserver 114.114.114.114 --gateway 172.16.1.1 \
  --subnet-range 172.16.1.0/24 selfservice
```

创建路由

```bash
openstack router create router
```

内部网络添加路由

```bash
neutron router-interface-add router selfservice
```

路由设置外部网关

```bash
neutron router-gateway-set router provider
```

## Dashboard

### 安装配置软件

[all]

安装软件

```bash
yum install -y openstack-dashboard
```

配置文件

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

重新启动 Web 服务器和会话存储服务

```bash
systemctl restart httpd.service memcached.service
systemctl status httpd.service memcached.service
```

### 配置 Haproxy

[all]

配置 haproxy 配置文件

```bash
cat << EOF >> /etc/haproxy/haproxy.cfg
listen dashboard_cluster
  bind 192.168.2.20:80
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 192.168.2.21:80 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:80 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:80 check inter 2000 rise 2 fall 5

EOF
```

重启`haproxy`

[controller1]

```bash
pcs resource restart haproxy
```

防火墙放行 80 端口

[all]

```bash
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
```

验证

打开浏览器，访问：http://192.168.2.20/dashboard/

域：default
用户名：admin
密码：ADMIN_PASS

## Block Storage service

### 安装控制部分

#### 配置 Haproxy

[all]

配置 haproxy 配置文件

```bash
cat << EOF >> /etc/haproxy/haproxy.cfg
listen cinder_api_cluster
  bind 192.168.2.20:8776
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
  server controller1 192.168.2.21:8776 check inter 2000 rise 2 fall 5
  server controller2 192.168.2.22:8776 check inter 2000 rise 2 fall 5
  server controller3 192.168.2.23:8776 check inter 2000 rise 2 fall 5

EOF
```

重启`haproxy`

[controller1]

```bash
pcs resource restart haproxy
```

#### 配置数据库

[controller1]

连接数据库

```bash
mysql -u root -p
```

创建`cinder`,并赋予`cinder`用户权限,刷新权限并推出

```sql
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY 'CINDER_DBPASS';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY 'CINDER_DBPASS';
FLUSH PRIVILEGES;
EXIT;
```

#### 创建用户和端点

[controller1]

加载`admin`凭证

```bash
. ~/admin-openrc
```

创建`cinder`用户

```bash
openstack user create --domain default --password CINDER_PASS cinder
```

将`admin`角色添加到`cinder`用户

```bash
openstack role add --project service --user cinder admin
```

创建`cinder`和`cinderv2`服务实体

```bash
openstack service create --name cinder \
  --description "OpenStack Block Storage" volume

openstack service create --name cinderv2 \
  --description "OpenStack Block Storage" volumev2
```

创建 Block Storage 服务 API 端点

```bash
openstack endpoint create --region RegionOne \
  volume public http://192.168.2.20:8776/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volume internal http://192.168.2.20:8776/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volume admin http://192.168.2.20:8776/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volumev2 public http://192.168.2.20:8776/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volumev2 internal http://192.168.2.20:8776/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volumev2 admin http://192.168.2.20:8776/v2/%\(tenant_id\)s
```

#### 安装和配置组件

[all]

安装软件包

```bash
yum install -y openstack-cinder
```

配置`/etc/cinder/cinder.conf`文件

[all]

```bash
openstack-config --set /etc/cinder/cinder.conf 'database' 'connection' 'mysql+pymysql://cinder:CINDER_DBPASS@controller/cinder'
openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'show_image_direct_url' 'True'
openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller1:5672,openstack:RABBIT_PASS@controller2:5672,openstack:RABBIT_PASS@controller3:5672'
openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'auth_strategy' 'keystone'
openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'auth_uri' 'http://controller:5000'
openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'auth_url' 'http://controller:35357'
openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'memcached_servers' 'controller1:11211,controller2:11211,controller3:11211'
openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'auth_type' 'password'
openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'project_domain_name' 'Default'
openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'user_domain_name' 'Default'
openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'project_name' 'service'
openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'username' 'cinder'
openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'password' 'CINDER_PASS'
openstack-config --set /etc/cinder/cinder.conf 'oslo_concurrency' 'lock_path' '/var/lib/cinder/tmp'
```

[controller1]

```bash
openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'my_ip' '192.168.2.21'
openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'osapi_volume_listen' '192.168.2.21'
```

[controller2]

```bash
openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'my_ip' '192.168.2.22'
openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'osapi_volume_listen' '192.168.2.22'
```

[controller3]

```bash
openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'my_ip' '192.168.2.23'
openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'osapi_volume_listen' '192.168.2.23'
```

填充块存储数据库

[controller1]

```bash
su -s /bin/sh -c "cinder-manage db sync" cinder
```

配置计算以使用块存储

[all]

```bash
openstack-config --set /etc/nova/nova.conf 'cinder' 'os_region_name' 'RegionOne'
```

重新启动 Compute API 服务

[all]

```bash
systemctl restart openstack-nova-api.service
```

启动 Block Storage 服务并将其配置为在系统引导时启动

[all]

```bash
systemctl restart openstack-cinder-api.service openstack-cinder-scheduler.service
systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
systemctl status openstack-cinder-api.service openstack-cinder-scheduler.service
```

### 对接 Ceph 存储

创建存储池

[controller1]

```bash
ceph osd pool create volumes 128
ceph osd pool create vms 128
```

创建 Cinder 用户

[controller1]

```bash
ceph auth get-or-create client.cinder mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=vms, allow rx pool=images'
```

client.cinder 的密钥环复制到适当的节点，并更改所有权

[controller1]

```bash
ceph auth get-or-create client.cinder | ssh controller1 sudo tee /etc/ceph/ceph.client.cinder.keyring
ssh controller1 sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring

ceph auth get-or-create client.cinder | ssh controller2 sudo tee /etc/ceph/ceph.client.cinder.keyring
ssh controller2 sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring

ceph auth get-or-create client.cinder | ssh controller3 sudo tee /etc/ceph/ceph.client.cinder.keyring
ssh controller3 sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring
```

在 nova-compute 的节点上创建一个密钥的临时副本

[all]

```bash
ceph auth get-key client.cinder > ~/client.cinder.key
```

在计算节点上把密钥加进 libvirt 、然后删除临时副本

[all]

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

配置`/etc/cinder/cinder.conf`文件

[all]

```bash
openstack-config --set  /etc/cinder/cinder.conf 'DEFAULT' 'enabled_backends' 'ceph'
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
```

配置计算`/etc/nova/nova.conf`文件

[all]

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

启用 rbd cache 功能

[all]

```bash
echo '
[client]
rbd cache = true
rbd cache writethrough until flush = true
admin socket = /var/run/ceph/guests/$cluster-$type.$id.$pid.$cctid.asok
log file = /var/log/qemu/qemu-guest-$pid.log
rbd concurrent management ops = 20

[client.cinder]
keyring = /etc/ceph/ceph.client.cinder.keyring
' >> /etc/ceph/ceph.conf

mkdir -p /var/run/ceph/guests/ /var/log/qemu/
chown qemu:libvirt /var/run/ceph/guests/ /var/log/qemu/

systemctl restart ceph.target
```

配置热迁移

[all]

```bash
sed -i 's/\(^\|\#\)listen_tls.\?=.*/listen_tls = 0/g' /etc/libvirt/libvirtd.conf
sed -i 's/\(^\|\#\)listen_tcp.\?=.*/listen_tcp = 1/g' /etc/libvirt/libvirtd.conf
sed -i 's/\(^\|\#\)tcp_port.\?=.*/tcp_port = \"16509\"/g' /etc/libvirt/libvirtd.conf
sed -i 's/\(^\|\#\)listen_addr.\?=.*/listen_addr = \"192\.168\.2\.243\"/g' /etc/libvirt/libvirtd.conf
sed -i 's/\(^\|\#\)auth_tcp.\?=.*/auth_tcp = \"none\"/g' /etc/libvirt/libvirtd.conf

sed -i 's/\(^\|\#\)LIBVIRTD_ARGS.\?=.*/LIBVIRTD_ARGS = \"--listen\"/g' /etc/sysconfig/libvirtd
```

重启计算节点

[all]

```bash
systemctl restart libvirtd.service openstack-nova-compute.service
systemctl status libvirtd.service openstack-nova-compute.service
```

重启 cinder-volume

[all]

```bash
systemctl restart openstack-cinder-volume.service
systemctl enable openstack-cinder-volume.service
systemctl status openstack-cinder-volume.service
```
