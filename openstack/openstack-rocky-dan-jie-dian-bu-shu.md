# OpenStack Rocky 单节点部署

## 环境

- 操作系统

  CentOS 7.6.1810

- 网络

  ens33: 192.168.200.11

  ens37: 未配置 IP

- 硬盘

  /dev/sda: 100G 系统盘

  /dev/sdb: 100G 空闲盘

- 主机名

  controller

## 配置主机名

```bash
hostnamectl set-hostname controller
hostname
```

## 配置 hosts 文件

```bash
echo '192.168.200.11    controller' >> /etc/hosts
ping controller -c 4
```

## 关闭 SeLinux

```bash
sed -i  's/^SELINUX=.\+$/SELINUX=permissive/g' /etc/selinux/config
egrep '^SELINUX=.+$' /etc/selinux/config
setenforce 0
getenforce
```

## 安装 openstack 包

```bash
yum install -y centos-release-openstack-rocky && \
yum install -y python-openstackclient openstack-utils && \
yum install -y openstack-selinux && \
yum -y upgrade && \
reboot
```

## 安装配置 MariaDB

- 安装软件包

  ```bash
  yum install -y mariadb mariadb-server python2-PyMySQL
  ```

- 配置数据库

  ```bash
  cat > /etc/my.cnf.d/openstack.cnf << EOF
  [mysqld]
  bind-address = 0.0.0.0

  default-storage-engine = innodb
  innodb_file_per_table = on
  max_connections = 4096
  collation-server = utf8_general_ci
  character-set-server = utf8
  EOF
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart mariadb.service && \
  systemctl status mariadb.service && \
  systemctl enable mariadb.service
  ```

- 数据库安全设置

  ```bash
  mysql_secure_installation
  ```

## 安装配置 RabbitMQ

- 安装软件包

  ```bash
  yum install -y rabbitmq-server
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart rabbitmq-server.service && \
  systemctl status rabbitmq-server.service && \
  systemctl enable rabbitmq-server.service
  ```

- 创建`openstack`用户

  ```bash
  rabbitmqctl add_user openstack RABBIT_PASS
  ```

- 配置用户权限

  ```bash
  rabbitmqctl set_permissions openstack ".*" ".*" ".*"
  ```

## 安装配置 Memcached

- 安装软件包

  ```bash
  yum install -y memcached python-memcached
  ```

- 修改配置文件

  ```bash
  sed -i 's/OPTIONS=.\+/OPTIONS="-l 0.0.0.0"/g' /etc/sysconfig/memcached
  egrep 'OPTIONS=.+' /etc/sysconfig/memcached
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart memcached.service && \
  systemctl status memcached.service && \
  systemctl enable memcached.service
  ```

## 安装配置 Etcd

- 安装软件包

  ```bash
  yum install -y etcd
  ```

- 配置 Etcd

  ```bash
  sed -i 's/\(^\|#\)ETCD_DATA_DIR=.\+/ETCD_DATA_DIR="\/var\/lib\/etcd\/default\.etcd"/g' /etc/etcd/etcd.conf
  sed -i 's/\(^\|#\)ETCD_LISTEN_PEER_URLS=.\+/ETCD_LISTEN_PEER_URLS="http:\/\/192.168.200.11:2380"/g' /etc/etcd/etcd.conf
  sed -i 's/\(^\|#\)ETCD_LISTEN_CLIENT_URLS=.\+/ETCD_LISTEN_CLIENT_URLS="http:\/\/192.168.200.11:2379"/g' /etc/etcd/etcd.conf
  sed -i 's/\(^\|#\)ETCD_NAME=.\+/ETCD_NAME="controller"/g' /etc/etcd/etcd.conf
  sed -i 's/\(^\|#\)ETCD_INITIAL_ADVERTISE_PEER_URLS=.\+/ETCD_INITIAL_ADVERTISE_PEER_URLS="http:\/\/192.168.200.11:2380"/g' /etc/etcd/etcd.conf
  sed -i 's/\(^\|#\)ETCD_ADVERTISE_CLIENT_URLS=.\+/ETCD_ADVERTISE_CLIENT_URLS="http:\/\/192.168.200.11:2379"/g' /etc/etcd/etcd.conf
  sed -i 's/\(^\|#\)ETCD_INITIAL_CLUSTER=.\+/ETCD_INITIAL_CLUSTER="controller=http:\/\/192.168.200.11:2380"/g' /etc/etcd/etcd.conf
  sed -i 's/\(^\|#\)ETCD_INITIAL_CLUSTER_TOKEN=.\+/ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"/g' /etc/etcd/etcd.conf
  sed -i 's/\(^\|#\)ETCD_INITIAL_CLUSTER_STATE=.\+/ETCD_INITIAL_CLUSTER_STATE="new"/g' /etc/etcd/etcd.conf
  grep -vE '^#' /etc/etcd/etcd.conf
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart etcd && \
  systemctl status etcd && \
  systemctl enable etcd
  ```

## 安装配置 Keystone

- 登录数据库

  ```bash
  mysql
  ```

- 创建数据库及用户

  ```sql
  CREATE DATABASE keystone;
  GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
  GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
  FLUSH PRIVILEGES;
  EXIT;
  ```

- 安装软件包

  ```bash
  yum install -y openstack-keystone httpd mod_wsgi
  ```

- 修改配置文件

  ```bash
  # 配置数据连接
  openstack-config --set /etc/keystone/keystone.conf 'database' 'connection' 'mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone'
  # 配置token为Fernet令牌
  openstack-config --set /etc/keystone/keystone.conf 'token' 'provider' 'fernet'
  ```

- 填充数据库

  ```bash
  su -s /bin/sh -c "keystone-manage db_sync" keystone
  ```

- 初始化 Fernet 令牌

  ```bash
  keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
  keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
  ```

- 引导 Identity 服务

  ```bash
  keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
    --bootstrap-admin-url http://192.168.200.11:5000/v3/ \
    --bootstrap-internal-url http://192.168.200.11:5000/v3/ \
    --bootstrap-public-url http://192.168.200.11:5000/v3/ \
    --bootstrap-region-id RegionOne
  ```

- 创建`/usr/share/keystone/wsgi-keystone.conf`链接

  ```bash
  ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart httpd.service && \
  systemctl status httpd.service && \
  systemctl enable httpd.service
  ```

- 配置管理帐户

  ```bash
  export OS_USERNAME=admin
  export OS_PASSWORD=ADMIN_PASS
  export OS_PROJECT_NAME=admin
  export OS_USER_DOMAIN_NAME=Default
  export OS_PROJECT_DOMAIN_NAME=Default
  export OS_AUTH_URL=http://controller:5000/v3
  export OS_IDENTITY_API_VERSION=3
  ```

- 创建`service`项目

  ```bash
  openstack project create --domain default \
    --description "Service Project" service
  ```

- `OS_AUTH_URL`和`OS_PASSWORD`环境变量

  ```bash
  unset OS_AUTH_URL OS_PASSWORD
  ```

- 获取 Token

  ```bash
   openstack --os-auth-url http://controller:5000/v3 \
    --os-project-domain-name Default --os-user-domain-name Default \
    --os-project-name admin --os-username admin --os-password ADMIN_PASS token issue
  ```

- 创建`admin-openrc`脚本

  ```bash
  cat > ~/admin-openrc << EOF
  export OS_PROJECT_DOMAIN_NAME=Default
  export OS_USER_DOMAIN_NAME=Default
  export OS_PROJECT_NAME=admin
  export OS_USERNAME=admin
  export OS_PASSWORD=ADMIN_PASS
  export OS_AUTH_URL=http://controller:5000/v3
  export OS_IDENTITY_API_VERSION=3
  export OS_IMAGE_API_VERSION=2
  EOF
  ```

- 使用脚本获取 Token

  ```bash
  . ~/admin-openrc
  openstack token issue
  ```

## 安装配置 Glance

- 登录数据库

  ```bash
  mysql
  ```

- 创建数据库及用户

  ```sql
  CREATE DATABASE glance;
  GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
    IDENTIFIED BY 'GLANCE_DBPASS';
  GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
    IDENTIFIED BY 'GLANCE_DBPASS';
  FLUSH PRIVILEGES;
  EXIT;
  ```

- 创建`glance`用户，添加到`service`项目并设置`admin`权限

  ```bash
  openstack user create --domain default --password GLANCE_PASS glance
  openstack role add --project service --user glance admin
  ```

- 创建`glance`服务

  ```bash
   openstack service create --name glance \
    --description "OpenStack Image" image
  ```

- 创建 api 端点

  ```bash
  openstack endpoint create --region RegionOne \
    image public http://192.168.200.11:9292
  openstack endpoint create --region RegionOne \
    image internal http://192.168.200.11:9292
  openstack endpoint create --region RegionOne \
    image admin http://192.168.200.11:9292
  ```

- 安装软件包

  ```bash
  yum install -y openstack-glance
  ```

- 修改配置文件

  ```bash
  # 配置数据库连接
  openstack-config --set /etc/glance/glance-api.conf 'database' 'connection' 'mysql+pymysql://glance:GLANCE_DBPASS@controller/glance'
  # 配置认证信息
  openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'project_domain_name' 'Default'
  openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'user_domain_name' 'Default'
  openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'username' 'glance'
  openstack-config --set /etc/glance/glance-api.conf 'keystone_authtoken' 'password' 'GLANCE_PASS'
  openstack-config --set /etc/glance/glance-api.conf 'paste_deploy' 'flavor' 'keystone'
  # 配置镜像存储
  openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'stores' 'file,http'
  openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'default_store' 'file'
  openstack-config --set /etc/glance/glance-api.conf 'glance_store' 'filesystem_store_datadir' '/var/lib/glance/images/'

  # 配置数据库连接
  openstack-config --set /etc/glance/glance-registry.conf 'database' 'connection' 'mysql+pymysql://glance:GLANCE_DBPASS@controller/glance'
  # 配置认证信息
  openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'project_domain_name' 'Default'
  openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'user_domain_name' 'Default'
  openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'username' 'glance'
  openstack-config --set /etc/glance/glance-registry.conf 'keystone_authtoken' 'password' 'GLANCE_PASS'
  openstack-config --set /etc/glance/glance-registry.conf 'paste_deploy' 'flavor' 'keystone'
  ```

- 填充数据库

  ```bash
  su -s /bin/sh -c "glance-manage db_sync" glance
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart openstack-glance-api.service \
    openstack-glance-registry.service
  systemctl status openstack-glance-api.service \
    openstack-glance-registry.service
  systemctl enable openstack-glance-api.service \
    openstack-glance-registry.service
  ```

- 加载预置元数据

  ```bash
  glance-manage db_load_metadefs
  ```

- 验证

  ```bash
  # 下载cirrso镜像
  wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
  # 上传镜像
  openstack image create "cirros" \
  --file cirros-0.4.0-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
  # 查看镜像列表
  openstack image list
  ```

## 安装配置 Compute 服务-控制节点

- 登录数据库

  ```bash
  mysql
  ```

- 创建数据库及用户

  ```sql
  CREATE DATABASE nova_api;
  CREATE DATABASE nova;
  CREATE DATABASE nova_cell0;
  CREATE DATABASE placement;
  GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
    IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
    IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
    IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
    IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
    IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
    IDENTIFIED BY 'NOVA_DBPASS';
  GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' \
    IDENTIFIED BY 'PLACEMENT_DBPASS';
  GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' \
    IDENTIFIED BY 'PLACEMENT_DBPASS';
  FLUSH PRIVILEGES;
  EXIT;
  ```

- 创建`nova`用户，添加到`service`项目并设置`admin`权限

  ```bash
  openstack user create --domain default --password NOVA_PASS nova
  openstack role add --project service --user nova admin
  ```

- 创建`nova`服务

  ```bash
  openstack service create --name nova \
    --description "OpenStack Compute" compute
  ```

- 创建 Api 端点

  ```bash
  openstack endpoint create --region RegionOne \
    compute public http://192.168.200.11:8774/v2.1
  openstack endpoint create --region RegionOne \
    compute internal http://192.168.200.11:8774/v2.1
  openstack endpoint create --region RegionOne \
    compute admin http://192.168.200.11:8774/v2.1
  ```

- 创建`placement`用户，添加到`service`项目并设置`admin`权限

  ```bash
  openstack user create --domain default --password PLACEMENT_PASS placement
  openstack role add --project service --user placement admin
  ```

- 创建`placement`服务

  ```bash
   openstack service create --name placement \
    --description "Placement API" placement
  ```

- 创建`Placement API` 端点

  ```bash
  openstack endpoint create --region RegionOne \
    placement public http://192.168.200.11:8778
  openstack endpoint create --region RegionOne \
    placement internal http://192.168.200.11:8778
  openstack endpoint create --region RegionOne \
    placement admin http://192.168.200.11:8778
  ```

- 安装软件包

  ```bash
  yum install -y openstack-nova-api openstack-nova-conductor \
    openstack-nova-console openstack-nova-novncproxy \
    openstack-nova-scheduler openstack-nova-placement-api
  ```

- 修改配置文件

  ```bash
  #启用 compute 和metadat api
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'enabled_apis' 'osapi_compute,metadata'
  # 配置数据库连接
  openstack-config --set /etc/nova/nova.conf 'api_database' 'connection' 'mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api'
  openstack-config --set /etc/nova/nova.conf 'database' 'connection' 'mysql+pymysql://nova:NOVA_DBPASS@controller/nova'
  openstack-config --set /etc/nova/nova.conf 'placement_database' 'connection' 'mysql+pymysql://placement:PLACEMENT_DBPASS@controller/placement'
  # 配置RabbitMQ连接
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller'
  # 配置认证信息
  openstack-config --set /etc/nova/nova.conf 'api' 'auth_strategy' 'keystone'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000/v3'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_domain_name' 'Default'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'user_domain_name' 'Default'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'username' 'nova'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'password' 'NOVA_PASS'
  # 配置本节点管理IP
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' '192.168.200.11'
  #启用网络服务
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'use_neutron' 'true'
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'firewall_driver' 'nova.virt.firewall.NoopFirewallDriver'
  # 配置 VNC代理
  openstack-config --set /etc/nova/nova.conf 'vnc' 'enabled' 'true'
  openstack-config --set /etc/nova/nova.conf 'vnc' 'server_listen' '$my_ip'
  openstack-config --set /etc/nova/nova.conf 'vnc' 'server_proxyclient_address' '$my_ip'
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
  openstack-config --set /etc/nova/nova.conf 'placement' 'password' 'PLACEMENT_PASS'
  ```

- 修改配置文件

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
  ```

- 重启 httpd 服务

  ```bash
  systemctl restart httpd
  ```

- 填充`nova-api`和`placement`数据库

  ```bash
  su -s /bin/sh -c "nova-manage api_db sync" nova
  ```

- 注册`cell0`数据库

  ```bash
  su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
  ```

- 创建`cell1`单元格

  ```bash
  su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
  ```

- 填充`nova`数据库

  ```bash
  su -s /bin/sh -c "nova-manage db sync" nova
  ```

- 验证 nova `cell0`和`cell1`是否正确注册

  ```bash
  su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart openstack-nova-api.service \
    openstack-nova-consoleauth openstack-nova-scheduler.service \
    openstack-nova-conductor.service openstack-nova-novncproxy.service
  systemctl status openstack-nova-api.service \
    openstack-nova-consoleauth openstack-nova-scheduler.service \
    openstack-nova-conductor.service openstack-nova-novncproxy.service
  systemctl enable openstack-nova-api.service \
    openstack-nova-consoleauth openstack-nova-scheduler.service \
    openstack-nova-conductor.service openstack-nova-novncproxy.service
  ```

## 安装配置 Compute 服务-计算节点

- 安装软件包

  ```bash
  yum install -y openstack-nova-compute
  ```

- 修改配置文件

  ```bash
  #启用 compute 和metadat api
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'enabled_apis' 'osapi_compute,metadata'
  # 配置RabbitMQ连接
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller'
  # 配置认证信息
  openstack-config --set /etc/nova/nova.conf 'api' 'auth_strategy' 'keystone'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000/v3'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_domain_name' 'Default'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'user_domain_name' 'Default'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'username' 'nova'
  openstack-config --set /etc/nova/nova.conf 'keystone_authtoken' 'password' 'NOVA_PASS'
  # 配置本节点管理IP
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'my_ip' '192.168.200.11'
  #启用网络服务
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'use_neutron' 'true'
  openstack-config --set /etc/nova/nova.conf 'DEFAULT' 'firewall_driver' 'nova.virt.firewall.NoopFirewallDriver'
  # 配置 VNC代理
  openstack-config --set /etc/nova/nova.conf 'vnc' 'enabled' 'true'
  openstack-config --set /etc/nova/nova.conf 'vnc' 'server_listen' '0.0.0.0'
  openstack-config --set /etc/nova/nova.conf 'vnc' 'server_proxyclient_address' '$my_ip'
  openstack-config --set /etc/nova/nova.conf 'vnc' 'novncproxy_base_url' 'http://192.168.200.11:6080/vnc_auto.html'
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
  openstack-config --set /etc/nova/nova.conf 'placement' 'password' 'PLACEMENT_PASS'
  ```

- 配置 qemu 虚拟化

  ```bash
  openstack-config --set /etc/nova/nova.conf 'libvirt' 'virt_type' 'qemu'
  openstack-config --set /etc/nova/nova.conf 'libvirt' 'cpm_mode' 'none'
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart libvirtd.service openstack-nova-compute.service
  systemctl status libvirtd.service openstack-nova-compute.service
  systemctl enable libvirtd.service openstack-nova-compute.service
  ```

- 将计算节点添加到`cell `数据库

  ```bash
  openstack compute service list --service nova-compute
  su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
  ```

- 验证

  ```bash
  openstack compute service list
  nova-status upgrade check
  ```

## 安装配置 Neutron 控制节点

- 登录数据库

  ```bash
  mysql
  ```

- 创建数据库及用户

  ```sql
  CREATE DATABASE neutron;
  GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
    IDENTIFIED BY 'NEUTRON_DBPASS';
  GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
    IDENTIFIED BY 'NEUTRON_DBPASS';
  FLUSH PRIVILEGES;
  EXIT;
  ```

- 创建`neutron`用户，添加到`service`项目并设置`admin`权限。

  ```bash
  openstack user create --domain default --password NEUTRON_PASS neutron
  openstack role add --project service --user neutron admin
  ```

- 创建`neutron`项目

  ```bash
  openstack service create --name neutron \
    --description "OpenStack Networking" network
  ```

- 创建网络服务端点

  ```bash
  openstack endpoint create --region RegionOne \
    network public http://192.168.200.11:9696
  openstack endpoint create --region RegionOne \
    network internal http://192.168.200.11:9696
  openstack endpoint create --region RegionOne \
    network admin http://192.168.200.11:9696
  ```

- 安装软件包

  ```bash
  yum install -y openstack-neutron openstack-neutron-ml2 \
    openstack-neutron-linuxbridge ebtables
  ```

- 修改配置文件

  ```bash
  # 配置数据库连接
  openstack-config --set /etc/neutron/neutron.conf 'database' 'connection' 'mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron'
  # 启用模块化第2层（ML2）插件，路由器服务和重叠的IP地址
  openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'core_plugin' 'ml2'
  openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'service_plugins' 'router'
  openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'allow_overlapping_ips' 'true'
  # 配置RabbitMQ 消息队列访问
  openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller'
  # 配置身份服务访问
  openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'auth_strategy' 'keystone'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_domain_name' 'Default'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'user_domain_name' 'Default'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'username' 'neutron'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'password' 'NEUTRON_PASS'
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
  openstack-config --set /etc/neutron/neutron.conf 'nova' 'password' 'NOVA_PASS'
  # 配置锁定路径
  openstack-config --set /etc/neutron/neutron.conf 'oslo_concurrency' 'lock_path' '/var/lib/neutron/tmp'
  ####
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
  ###
  # 将提供者虚拟网络映射到提供者物理网络接口
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'linux_bridge' 'physical_interface_mappings' 'provider:ens37'
  # 启用VXLAN重叠网络，配置处理覆盖网络的物理网络接口的IP地址，并启用第2层填充
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'enable_vxlan' 'true'
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' '192.168.200.11'
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'l2_population' 'true'
  # 启用安全组并配置Linux网桥iptables防火墙驱动程序
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'enable_security_group' 'true'
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'firewall_driver' 'neutron.agent.linux.iptables_firewall.IptablesFirewallDriver'
  ###
  # 配置Linux桥接接口驱动程序和外部网桥
  openstack-config --set /etc/neutron/l3_agent.ini 'DEFAULT' 'interface_driver' 'linuxbridge'
  ###
  # 配置Linux桥接接口驱动程序，Dnsmasq DHCP驱动程序，并启用隔离的元数据，以便提供商网络上的实例可以通过网络访问元数据
  openstack-config --set /etc/neutron/dhcp_agent.ini 'DEFAULT' 'interface_driver' 'linuxbridge'
  openstack-config --set /etc/neutron/dhcp_agent.ini 'DEFAULT' 'dhcp_driver' 'neutron.agent.linux.dhcp.Dnsmasq'
  openstack-config --set /etc/neutron/dhcp_agent.ini 'DEFAULT' 'enable_isolated_metadata' 'true'
  ###
  # 配置元数据主机和共享密钥
  openstack-config --set /etc/neutron/metadata_agent.ini 'DEFAULT' 'nova_metadata_host' 'controller'
  openstack-config --set /etc/neutron/metadata_agent.ini 'DEFAULT' 'metadata_proxy_shared_secret' 'METADATA_SECRET'
  ###
  # 配置访问参数，启用元数据代理并配置密码
  openstack-config --set /etc/nova/nova.conf 'neutron' 'url' 'http://controller:9696'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_type' 'password'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'project_domain_name' 'default'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'user_domain_name' 'default'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'region_name' 'RegionOne'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'project_name' 'service'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'username' 'neutron'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'password' 'NEUTRON_PASS'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'service_metadata_proxy' 'true'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'metadata_proxy_shared_secret' 'METADATA_SECRET'
  ```

- 创建初始化脚本软链

  ```bash
  ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
  ```

- 填充数据库

  ```bash
  su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
    --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
  ```

- 重新启动 Compute API 服务

  ```bash
  systemctl restart openstack-nova-api.service
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart neutron-server.service \
    neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
    neutron-metadata-agent.service neutron-l3-agent.service
  systemctl status neutron-server.service \
    neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
    neutron-metadata-agent.service neutron-l3-agent.service
  systemctl enable neutron-server.service \
    neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
    neutron-metadata-agent.service neutron-l3-agent.service
  ```

## 安装配置 Neutron 服务-计算节点

- 安装软件包

  ```bash
  yum install -y openstack-neutron-linuxbridge ebtables ipset
  ```

- 修改配置文件

  ```bash
  # 配置RabbitMQ 消息队列访问
  openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller'
  # 配置身份服务访问
  openstack-config --set /etc/neutron/neutron.conf 'DEFAULT' 'auth_strategy' 'keystone'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_domain_name' 'Default'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'user_domain_name' 'Default'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'username' 'neutron'
  openstack-config --set /etc/neutron/neutron.conf 'keystone_authtoken' 'password' 'NEUTRON_PASS'
  # 配置锁定路径
  openstack-config --set /etc/neutron/neutron.conf 'oslo_concurrency' 'lock_path' '/var/lib/neutron/tmp'
  ###
  # 将提供者虚拟网络映射到提供者物理网络接口
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'linux_bridge' 'physical_interface_mappings' 'provider:ens37'
  # 启用VXLAN重叠网络，配置处理覆盖网络的物理网络接口的IP地址，并启用第2层填充
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'enable_vxlan' 'true'
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'local_ip' '192.168.200.11'
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'vxlan' 'l2_population' 'true'
  # 启用安全组并配置Linux网桥iptables防火墙驱动程序
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'enable_security_group' 'true'
  openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini 'securitygroup' 'firewall_driver' 'neutron.agent.linux.iptables_firewall.IptablesFirewallDriver'
  ###
  # 配置Compute服务以使用Networking服务
  openstack-config --set /etc/nova/nova.conf 'neutron' 'url' 'http://controller:9696'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'auth_type' 'password'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'project_domain_name' 'default'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'user_domain_name' 'default'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'region_name' 'RegionOne'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'project_name' 'service'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'username' 'neutron'
  openstack-config --set /etc/nova/nova.conf 'neutron' 'password' 'NEUTRON_PASS'
  ```

- 重新启动 Compute 服务

  ```bash
  systemctl restart openstack-nova-compute.service
  ```

- 启动服务，检查状态并设置开机自启

  ```bash
  systemctl restart neutron-linuxbridge-agent.service
  systemctl status neutron-linuxbridge-agent.service
  systemctl enable neutron-linuxbridge-agent.service
  ```

- 检验

  ```bash
  openstack extension list --network
  openstack network agent list
  ```

- 创建外部网络

  ```bash
  openstack network create  --share --external \
    --provider-physical-network provider \
    --provider-network-type flat provider
  ```

- 创建外部网络子网

  ```bash
  openstack subnet create --network provider \
    --allocation-pool start=192.168.5.10,end=192.168.5.20 \
    --dns-nameserver 114.114.114.114 --dns-nameserver 8.8.8.8 \
    --gateway 192.168.5.1 \
    --subnet-range 192.168.5.0/24 provider
  ```

- 创建内部网络

  ```bash
  openstack network create selfservice
  ```

- 创建内部网络子网

  ```bash
  openstack subnet create --network selfservice \
    --dns-nameserver 114.114.114.114 --dns-nameserver 8.8.8.8 \
    --gateway 172.16.1.1 \
    --subnet-range 172.16.1.0/24 selfservice
  ```

- 创建路由

  ```bash
  openstack router create router
  ```

- 内部网络添加路由

  ```bash
  neutron router-interface-add router selfservice
  ```

- 路由设置外部网关

  ```bash
  neutron router-gateway-set router provider
  ```

## 安装配置 Dashboard

- 安装软件

  ```bash
  yum install -y openstack-dashboard
  ```

- 配置文件

  ```bash
  cat >> /etc/openstack-dashboard/local_settings << EOF
  OPENSTACK_HOST = "controller"
  ALLOWED_HOSTS = ['*', ]
  SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

  CACHES = {
      'default': {
           'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
           'LOCATION': 'controller:11211',
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

- 重新启动 Web 服务器和会话存储服务

  ```bash
  systemctl restart httpd.service memcached.service
  systemctl status httpd.service memcached.service
  ```

## 安装配置 Cinder 服务-控制节点

- 登录数据库

  ```bash
  mysql
  ```

- 创建数据库及用户

  ```sql
  CREATE DATABASE cinder;
  GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
    IDENTIFIED BY 'CINDER_DBPASS';
  GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
    IDENTIFIED BY 'CINDER_DBPASS';
  FLUSH PRIVILEGES;
  EXIT;
  ```

- 创建`cinder`用户，添加到`service`项目并设置`admin`权限。

  ```bash
  openstack user create --domain default --password CINDER_PASS cinder
  openstack role add --project service --user cinder admin
  ```

- 创建`cinderv2`和`cinderv3`服务实体

  ```bash
  openstack service create --name cinderv2 \
    --description "OpenStack Block Storage" volumev2
  openstack service create --name cinderv3 \
    --description "OpenStack Block Storage" volumev3
  ```

- 创建`Block Storage`服务 API 端点

  ```bash
  openstack endpoint create --region RegionOne \
    volumev2 public http://192.168.200.11:8776/v2/%\(project_id\)s
  openstack endpoint create --region RegionOne \
    volumev2 internal http://192.168.200.11:8776/v2/%\(project_id\)s
  openstack endpoint create --region RegionOne \
    volumev2 admin http://192.168.200.11:8776/v2/%\(project_id\)s

  openstack endpoint create --region RegionOne \
    volumev3 public http://192.168.200.11:8776/v3/%\(project_id\)s
  openstack endpoint create --region RegionOne \
    volumev3 internal http://192.168.200.11:8776/v3/%\(project_id\)s
  openstack endpoint create --region RegionOne \
    volumev3 admin http://192.168.200.11:8776/v3/%\(project_id\)s
  ```

- 安装软件包

  ```bash
  yum install -y openstack-cinder
  ```

- 编辑配置文件

  ```bash
  # 配置数据库访问
  openstack-config --set /etc/cinder/cinder.conf 'database' 'connection' 'mysql+pymysql://cinder:CINDER_DBPASS@controller/cinder'
  # 配置RabbitMQ 消息队列访问
  openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller'
  # 配置身份服务访问
  openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'auth_strategy' 'keystone'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'project_domain_id' 'default'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'user_domain_id' 'default'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'username' 'cinder'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'password' 'CINDER_PASS'
  # 控制器节点的管理接口IP地址
  openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'my_ip' '192.168.200.11'
  # 配置锁定路径
  openstack-config --set /etc/cinder/cinder.conf 'oslo_concurrency' 'lock_path' '/var/lib/cinder/tmp'
  ##
  # 配置计算使用块存储
  openstack-config --set /etc/nova/nova.conf 'cinder' 'os_region_name' 'RegionOne'
  ```

- 填充数据库

  ```bash
  su -s /bin/sh -c "cinder-manage db sync" cinder
  ```

- 重新启动`Compute API`服务

  ```bash
  systemctl restart openstack-nova-api.service
  ```

- 启动`Block Storage`服务并将其配置为在系统引导时启动

  ```bash
  systemctl restart openstack-cinder-api.service openstack-cinder-scheduler.service
  systemctl status openstack-cinder-api.service openstack-cinder-scheduler.service
  systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
  ```

## 安装配置 Cinder 服务-存储节点

- 安装软件包

  ```bash
  yum install -y lvm2 device-mapper-persistent-data
  ```

- 启动 LVM 元数据服务并将其配置为在系统引导时启动

  ```bash
  systemctl restart lvm2-lvmetad.service
  systemctl status lvm2-lvmetad.service
  systemctl enable lvm2-lvmetad.service
  ```

- 创建 LVM 物理卷`/dev/sdb` 及卷组`cinder-volumes`

  ```bash
  pvcreate /dev/sdb
  vgcreate cinder-volumes /dev/sdb
  ```

- 修改配置文件`/etc/lvm/lvm.conf`

  ```properties
  filter = [ "a/sda/", "a/sdb/", "r/.*/"]
  ```

- 安装软件包

  ```bash
  yum install -y openstack-cinder targetcli python-keystone
  ```

- 修改配置文件

  ```bash
  # 配置数据库访问
  openstack-config --set /etc/cinder/cinder.conf 'database' 'connection' 'mysql+pymysql://cinder:CINDER_DBPASS@controller/cinder'
  # 配置RabbitMQ 消息队列访问
  openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller'
  # 配置身份服务访问
  openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'auth_strategy' 'keystone'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'project_domain_id' 'default'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'user_domain_id' 'default'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'username' 'cinder'
  openstack-config --set /etc/cinder/cinder.conf 'keystone_authtoken' 'password' 'CINDER_PASS'
  # 配置my_ip选项
  openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'my_ip' '192.168.200.11'
  # 使用LVM驱动程序
  openstack-config --set /etc/cinder/cinder.conf 'lvm' 'volume_driver' 'cinder.volume.drivers.lvm.LVMVolumeDriver'
  openstack-config --set /etc/cinder/cinder.conf 'lvm' 'volume_group' 'cinder-volumes'
  openstack-config --set /etc/cinder/cinder.conf 'lvm' 'iscsi_protocol' 'iscsi'
  openstack-config --set /etc/cinder/cinder.conf 'lvm' 'iscsi_helper' 'lioadm'
  # 启用LVM后端
  openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'enabled_backends' 'lvm'
  # 配置Image服务API的位置
  openstack-config --set /etc/cinder/cinder.conf 'DEFAULT' 'glance_api_servers' 'http://controller:9292'
  # 配置锁定路径
  openstack-config --set /etc/cinder/cinder.conf 'oslo_concurrency' 'lock_path' '/var/lib/cinder/tmp'
  ```

- 启动 Block Storage 卷服务（包括其依赖项）并将其配置为在系统引导时启动

  ```bash
  systemctl restart openstack-cinder-volume.service target.service
  systemctl status openstack-cinder-volume.service target.service
  systemctl enable openstack-cinder-volume.service target.service
  ```

- 验证

  ```bash
  openstack volume service list
  ```

## 安装配置 Manila-控制节点

- 登录数据库

  ```bash
  mysql
  ```

- 创建数据库并设置权限

```sql
CREATE DATABASE manila;
GRANT ALL PRIVILEGES ON manila.* TO 'manila'@'localhost' \
  IDENTIFIED BY 'MANILA_DBPASS';
GRANT ALL PRIVILEGES ON manila.* TO 'manila'@'%' \
  IDENTIFIED BY 'MANILA_DBPASS';
FLUSH PRIVILEGES;
EXIT;
```

- 创建`manila`用户,添加到`service`项目，并设置`admin`权限

  ```bash
  openstack user create --domain default --password MANILA_PASS manila
  openstack role add --project service --user manila admin
  ```

- 创建`manila`和`manilav2`服务实体

  ```bash
  openstack service create --name manila \
    --description "OpenStack Shared File Systems" share
  openstack service create --name manilav2 \
    --description "OpenStack Shared File Systems V2" sharev2
  ```

- 创建共享文件系统服务 API 端点

  ```bash
  openstack endpoint create --region RegionOne \
    share public http://192.168.200.11:8786/v1/%\(tenant_id\)s
  openstack endpoint create --region RegionOne \
    share internal http://192.168.200.11:8786/v1/%\(tenant_id\)s
  openstack endpoint create --region RegionOne \
    share admin http://192.168.200.11:8786/v1/%\(tenant_id\)s

  openstack endpoint create --region RegionOne \
    sharev2 public http://192.168.200.11:8786/v2/%\(tenant_id\)s
  openstack endpoint create --region RegionOne \
    sharev2 internal http://192.168.200.11:8786/v2/%\(tenant_id\)s
  openstack endpoint create --region RegionOne \
    sharev2 admin http://192.168.200.11:8786/v2/%\(tenant_id\)s
  ```

- 安装软件包

  ```bash
  yum install -y openstack-manila python-manilaclient
  ```

- 修改配置文件
  ```bash
  # 配置数据库访问
  openstack-config --set /etc/manila/manila.conf 'database' 'connection' 'mysql+pymysql://manila:MANILA_DBPASS@controller/manila'
  # 配置RabbitMQ 消息队列访问
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller'
  # 配置默认值
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'default_share_type'   'default_share_type'
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'share_name_template' 'share-%s'
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'rootwrap_config' '/etc/manila/rootwrap.conf'
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'api_paste_config' '/etc/manila/api-paste.ini'
  # 配置身份服务访问
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'auth_strategy' 'keystone'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'project_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'user_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'username' 'manila'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'password' 'MANILA_PASS'
  # 配置my_ip选项以使用控制器节点的管理接口IP地址
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'my_ip' '192.168.200.11'
  # 配置锁定路径
  openstack-config --set /etc/manila/manila.conf 'oslo_concurrency' 'lock_path' '/var/lib/manila/tmp'
  ```
- 填充数据库

```bash
su -s /bin/sh -c "manila-manage db sync" manila
```

- 启动共享文件系统服务并将其配置为在系统引导时启动

  ```bash
  systemctl restart openstack-manila-api.service openstack-manila-scheduler.service
  systemctl status openstack-manila-api.service openstack-manila-scheduler.service
  systemctl enable openstack-manila-api.service openstack-manila-scheduler.service
  ```

## 安装配置 Manila-共享节点

- 安装软件包

  ```bash
  yum install -y openstack-manila-share python2-PyMySQL openstack-neutron openstack-neutron-linuxbridge ebtables
  ```

- 修改配置文件

  ```bash
  # 配置数据库访问
  openstack-config --set /etc/manila/manila.conf 'database' 'connection' 'mysql+pymysql://manila:MANILA_DBPASS@controller/manila'
  # 配置RabbitMQ 消息队列访问
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'transport_url' 'rabbit://openstack:RABBIT_PASS@controller'
  # 配置默认值
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'default_share_type' 'default_share_type'
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'rootwrap_config' '/etc/manila/rootwrap.conf'
  # 配置身份服务访问
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'auth_strategy' 'keystone'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'auth_type' 'password'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'project_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'user_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'username' 'manila'
  openstack-config --set /etc/manila/manila.conf 'keystone_authtoken' 'password' 'MANILA_PASS'
  # 配置my_ip选项以使用控制器节点的管理接口IP地址
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'my_ip' '192.168.2.10'
  # 配置锁定路径
  openstack-config --set /etc/manila/manila.conf 'oslo_concurrency' 'lock_path' '/var/lib/manila/tmp'
  # 启用generic驱动程序和NFS协议
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'enabled_share_backends' 'generic'
  openstack-config --set /etc/manila/manila.conf 'DEFAULT' 'enabled_share_protocols' 'NFS'
  # 启用neutron服务的身份验证
  openstack-config --set /etc/manila/manila.conf 'neutron' 'url' 'http://controller:9696'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'auth_type' 'password'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'project_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'user_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'region_name' 'RegionOne'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'project_name' 'service'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'username' 'neutron'
  openstack-config --set /etc/manila/manila.conf 'neutron' 'password' 'NEUTRON_PASS'
  # 启用nova服务的身份验证
  openstack-config --set /etc/manila/manila.conf 'nova' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'nova' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'nova' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/manila/manila.conf 'nova' 'auth_type' 'password'
  openstack-config --set /etc/manila/manila.conf 'nova' 'project_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'nova' 'user_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'nova' 'region_name' 'RegionOne'
  openstack-config --set /etc/manila/manila.conf 'nova' 'project_name' 'service'
  openstack-config --set /etc/manila/manila.conf 'nova' 'username' 'nova'
  openstack-config --set /etc/manila/manila.conf 'nova' 'password' 'NOVA_PASS'
  # 启用cinder服务的身份验证
  openstack-config --set /etc/manila/manila.conf 'cinder' 'www_authenticate_uri' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'cinder' 'auth_url' 'http://controller:5000'
  openstack-config --set /etc/manila/manila.conf 'cinder' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/manila/manila.conf 'cinder' 'auth_type' 'password'
  openstack-config --set /etc/manila/manila.conf 'cinder' 'project_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'cinder' 'user_domain_name' 'Default'
  openstack-config --set /etc/manila/manila.conf 'cinder' 'region_name' 'RegionOne'
  openstack-config --set /etc/manila/manila.conf 'cinder' 'project_name' 'service'
  openstack-config --set /etc/manila/manila.conf 'cinder' 'username' 'cinder'
  openstack-config --set /etc/manila/manila.conf 'cinder' 'password' 'CINDER_PASS'
  # 配置generic驱动程序
  openstack-config --set /etc/manila/manila.conf 'generic' 'share_backend_name' 'GENERIC'
  openstack-config --set /etc/manila/manila.conf 'generic' 'share_driver' 'manila.share.drivers.generic.GenericShareDriver'
  openstack-config --set /etc/manila/manila.conf 'generic' 'driver_handles_share_servers' 'True'
  openstack-config --set /etc/manila/manila.conf 'generic' 'service_instance_flavor_id' '100'
  openstack-config --set /etc/manila/manila.conf 'generic' 'service_image_name' 'manila-service-image'
  openstack-config --set /etc/manila/manila.conf 'generic' 'service_instance_user' 'manila'
  openstack-config --set /etc/manila/manila.conf 'generic' 'service_instance_password' 'manila'
  openstack-config --set /etc/manila/manila.conf 'generic' 'interface_driver' 'manila.network.linux.interface.BridgeInterfaceDriver'
  ```

- 启动服务并设置开机自启

  ```bash
  systemctl restart openstack-manila-share.service
  systemctl status openstack-manila-share.service
  systemctl enable openstack-manila-share.service
  ```

- 验证

  ```bash
  manila service-list
  ```

- 安装 Manila UI

```bash
yum install -y openstack-manila-ui
systemctl restart httpd memcached
```

## 安装配置 Octavia

- 登录数据库

  ```bash
  mysql
  ```

- 创建数据库及设置权限

  ```sql
  CREATE DATABASE octavia;
  GRANT ALL PRIVILEGES ON octavia.* TO 'octavia'@'localhost' IDENTIFIED BY 'OCTAVIA_DBPASS';
  GRANT ALL PRIVILEGES ON octavia.* TO 'octavia'@'%' IDENTIFIED BY 'OCTAVIA_DBPASS';
  FLUSH PRIVILEGES;
  ```

* 创建`octavia`用户,添加到`service`项目，并设置`admin`权限

  ```bash
  openstack user create --domain default --password OCTAVIA_PASS octavia
  openstack role add --project service --user octavia admin
  ```

* 创建`octavia`服务实体

  ```bash
  openstack service create --name octavia \
    --description "OpenStack Octavia" load-balancer
  ```

* 创建 API 端点

  ```bash
  openstack endpoint create --region RegionOne \
    octavia public "http://192.168.200.11:9876"
  openstack endpoint create --region RegionOne \
    octavia internal "http://192.168.200.11:9876"
  openstack endpoint create --region RegionOne \
    octavia admin "http://192.168.200.11:9876"
  ```

* 切换到 Octavia 用户

  ```bash
  export OS_PROJECT_DOMAIN_NAME=Default
  export OS_USER_DOMAIN_NAME=Default
  export OS_PROJECT_NAME=service
  export OS_USERNAME=octavia
  export OS_PASSWORD=OCTAVIA_PASS
  export OS_AUTH_URL=http://controller:5000/v3
  export OS_IDENTITY_API_VERSION=3
  export OS_IMAGE_API_VERSION=2

  ```

* 安装软件包

  ```bash
  yum install -y openstack-octavia-api.noarch openstack-octavia-common.noarch openstack-octavia-health-manager.noarch openstack-octavia-housekeeping.noarch openstack-octavia-worker.noarch python2-octaviaclient.noarch
  ```

* 修改配置文件

  ```bash
  # 设置api服务绑定的IP和端口
  openstack-config --set /etc/octavia/octavia.conf 'api_settings' 'bind_host' "192.168.200.11"
  openstack-config --set /etc/octavia/octavia.conf 'api_settings' 'bind_port' '9876'

  # 配置数据库连接
  openstack-config --set /etc/octavia/octavia.conf 'database' 'connection' "mysql+pymysql://octavia:OCTAVIA_DBPASS@controller/octavia"

  # 配置认证信息
  openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'project_domain_name' 'Default'
  openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'project_name' 'service'
  openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'user_domain_name' 'Default'
  openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'username' 'octavia'
  openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'password' "OCTAVIA_PASS"
  openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'auth_url' 'http://controller:5000/v3'
  openstack-config --set /etc/octavia/octavia.conf 'keystone_authtoken' 'auth_type' 'password'

  # 创建amphora虚拟机的对象
  openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'memcached_servers' 'controller:11211'
  openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'project_domain_name' 'Default'
  openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'project_name' 'service'
  openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'user_domain_name' 'Default'
  openstack-config --set /etc/octavia/octavia.conf 'service_auth' 'password' "OCTAVIA_PASS"
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
  openstack-config --set /etc/octavia/octavia.conf 'DEFAULT' 'transport_url' "rabbit://openstack:RABBIT_PASS@controller:5672"
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

* 创建和配置证书

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

  # 修改配置文件
  openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'client_cert' '/etc/octavia/certs/client.pem'
  openstack-config --set /etc/octavia/octavia.conf 'haproxy_amphora' 'server_ca' '/etc/octavia/certs/ca_01.pem'
  openstack-config --set /etc/octavia/octavia.conf 'certificates' 'ca_certificate' '/etc/octavia/certs/ca_01.pem'
  openstack-config --set /etc/octavia/octavia.conf 'certificates' 'ca_private_key' '/etc/octavia/certs/private/cakey.pem'
  openstack-config --set /etc/octavia/octavia.conf 'certificates' 'ca_private_key_passphrase' 'foobar'
  openstack-config --set /etc/octavia/octavia.conf 'certificates' 'server_certs_key_passphrase' 'insecure-key-do-not-use-this-key'
  ```

* 创建和配置 ssh key

  ```bash
  rm -rf /etc/octavia/.ssh/
  mkdir -m755 /etc/octavia/.ssh/
  ssh-keygen -b 2048 -t rsa -N '' -f /etc/octavia/.ssh/octavia_ssh_key
  # 上传ssh key
  openstack keypair create --public-key /etc/octavia/.ssh/octavia_ssh_key.pub octavia_ssh_key
  # 修改配置文件
  openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_ssh_key_name' 'octavia_ssh_key'

  ```

* 配置 amphora 镜像

  ```bash
  # 上传镜像
  openstack image create amphora-x64-haproxy --public --container-format=bare --disk-format qcow2 --file amphora-x64-haproxy.qcow2 --tag amphora
  # 获取变量
  image_id=$(openstack image list --property name=amphora-x64-haproxy -f value -c ID)
  owner_id=$(openstack image show ${image_id} -c owner -f value)
  # 修改配置文件
  openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_image_tag' 'amphora'
  openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_image_owner_id' "$owner_id"

  ```

* 配置机型

  ```bash
  # 创建机型
  openstack flavor create --id auto --ram 1024 --disk 3 --vcpus 1 --private m1.amphora -f value -c id
  amp_flavor_id=$(openstack flavor show m1.amphora -f value -c id)
  # 修改配置文件
  openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_flavor_id' "$amp_flavor_id"
  ```

* 创建和配置网络

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
  # 创建负载监控管理接口（绑定到物理机的接口)
  MGMT_PORT_ID=$(openstack port create --security-group lb-health-mgr-sec-grp --device-owner Octavia:health-mgr --host=$(hostname) -c id -f value --network lb-mgmt-net octavia-health-manager-listen-port)


  # 修改配置文件
  ## 配置引导网络
  OCTAVIA_AMP_NETWORK_ID=$(openstack network show lb-mgmt-net -f value -c id)
  openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_boot_network_list' "$OCTAVIA_AMP_NETWORK_ID"
  ## 配置监控管理
  MGMT_PORT_IP=$(openstack port show -f value -c fixed_ips $MGMT_PORT_ID | awk '{FS=",| "; gsub(",",""); gsub("'\''",""); for(i = 1; i <= NF; ++i) {if ($i ~ /^ip_address/) {n=index($i, "="); if (substr($i, n+1) ~ "\\.") print substr($i, n+1)}}}')
  openstack-config --set /etc/octavia/octavia.conf 'health_manager' 'controller_ip_port_list' "$MGMT_PORT_IP:5555"
  openstack-config --set /etc/octavia/octavia.conf 'health_manager' 'bind_ip' "$MGMT_PORT_IP"
  openstack-config --set /etc/octavia/octavia.conf 'health_manager' 'bind_port' '5555'
  ## 配置防火墙
  OCTAVIA_MGMT_SEC_GRP_ID=$(openstack security group show lb-mgmt-sec-grp -f value -c id)
  openstack-config --set /etc/octavia/octavia.conf 'controller_worker' 'amp_secgroup_list' "$OCTAVIA_MGMT_SEC_GRP_ID"


  ```

* 物理机绑定监控管理接口

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

* 设置文件权限

  ```bash
  chown -R octavia:octavia /etc/octavia/
  ```

* 填充数据库

  ```bash
  octavia-db-manage upgrade head
  ```

* 启动服务

  ```bash
  systemctl restart octavia-api.service
  systemctl restart octavia-worker.service
  systemctl restart octavia-housekeeping.service
  systemctl restart octavia-health-manager.service

  ```
