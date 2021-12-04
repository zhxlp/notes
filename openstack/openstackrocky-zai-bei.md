# OpenStack-Rocky 灾备

## 环境

* 操作系统: CentOS7.6
* OpenStack: Rocky
* Ceph: luminous
* 数据库集群：MariaDB Galera Cluster

> 为了便于区别 Openstack 集群和灾备服务器，将 OpenStack 集群服务器简称为 openstack，灾备服务器简称为 backup

## 部署灾备服务器

### ceph 备份

* backup 正常部署 ceph
*   ceph 开启日志功能

    RBD 需要启用`exclusive-lock`和`journaling`属性,可以手动创建时启用，也可以修改配置文件，创建时自动启用。

    \[all]

    ```bash
    # 修改配置文件,开启自动启动功能
    crudini --set /etc/ceph/ceph.conf 'global' 'rbd default features' '125'
    systemctl restart ceph.target
    ```
*   backup 创建 pool

    \[backup]

    ```bash
    ceph osd pool create images 64
    ceph osd pool create vms 64
    ceph osd pool create volumes 64
    ```
*   拷贝配置文件和密钥到 backup

    \[openstack]

    ```bash
    scp /etc/ceph/ceph.conf root@backup_ip:/etc/ceph/openstack.conf
    scp /etc/ceph/ceph.client.admin.keyring root@backup_ip:/etc/ceph/openstack.client.admin.keyring
    ```
*   修改 ceph 配置文件所有者

    ```bash
    chown ceph:ceph /etc/ceph/
    ```
*   pool 启用 mirror

    \[all]

    ```bash
    rbd mirror pool enable images pool
    rbd mirror pool enable vms pool
    rbd mirror pool enable volumes pool
    ```
*   添加 peer

    \[backup]

    ```bash
    rbd mirror pool peer add images client.admin@openstack
    rbd mirror pool peer add vms client.admin@openstack
    rbd mirror pool peer add volumes client.admin@openstack
    ```
*   防火墙放行 backup IP

    \[master]

    ```bash
    firewall-cmd --permanent --zone=trusted --add-source=192.168.2.10
    firewall-cmd --zone=trusted --add-source=192.168.2.10
    ```
*   启动`ceph-rbd-mirror`服务

    \[backup]

    ```bash
    yum install -y rbd-mirror
    systemctl start ceph-rbd-mirror@admin.service
    systemctl enable ceph-rbd-mirror@admin.service
    systemctl status ceph-rbd-mirror@admin.service
    ```
*   查看同步状态

    ```bash
    rbd mirror pool status images
    rbd mirror pool status vms
    rbd mirror pool status volumes
    ```

### MariaDB Galera Cluster 备份

*   安装软包

    \[backup]

    ```bash
    yum install -y centos-release-openstack-rocky
    yum install -y mariadb mariadb-galera-server mariadb-galera-common galera rsync
    ```
*   修改配置文件

    \[backup]

    ```bash
    cat > /etc/my.cnf.d/openstack.cnf << EOF
    [mysqld]
    bind-address = 0.0.0.0

    default-storage-engine = innodb
    innodb_file_per_table
    max_connections = 4096
    collation-server = utf8_general_ci
    character-set-server = utf8

    wsrep_provider = /usr/lib64/galera/libgalera_smm.so
    wsrep_cluster_address = "gcomm://controller1,controller2,controller3"
    wsrep_node_name = backup
    wsrep_node_address = 192.168.2.10

    EOF
    ```

    > 注意：根据环境修改 `wsrep_cluster_address` , `wsrep_node_name` , `wsrep_node_address`
*   启动服务

    \[backup]

    ```bash
    systemctl restart mariadb.service
    systemctl enabel mariadb.service
    systemctl status mariadb.service
    ```
*   检查同步

    \[backup]

    ```bash
    mysql -e 'SHOW STATUS LIKE "wsrep%";'
    ```

    ### 灾备恢复

    根据之前的网络架构，重新部署 OpenStack 集群，再进行灾备恢复

    > 注意：不要上传任何镜像，创建虚拟机和卷

    #### ceph 恢复
*   确认 images,vms,volumes 没有文件

    \[openstack]

    ```bash
    rbd ls images
    rbd ls vms
    rbd ls volumes
    ```
*   提升等级

    \[backup]

    ```bash
    rbd mirror pool promote --force images
    rbd mirror pool promote --force vms
    rbd mirror pool promote --force volumes
    ```
*   开启 ceph 日志功能

    \[openstack]

    ```bash
    # 修改配置文件,开启自动启动功能
    crudini --set /etc/ceph/ceph.conf 'global' 'rbd default features' '125'
    systemctl restart ceph.target
    ```
*   拷贝密钥和配置文件

    \[backup]

    ```bash
    scp /etc/ceph/ceph.conf root@openstack_ip:/etc/ceph/backup.conf
    scp /etc/ceph/ceph.client.admin.keyring root@openstack_ip:/etc/ceph/backup.client.admin.keyring
    ```

    *   修改 ceph 配置文件所有者

        \[openstack]

        ```bash
        chown ceph:ceph /etc/ceph/
        ```
    *   pool 启用 mirror

        \[openstack]

        ```bash
        rbd mirror pool enable images pool
        rbd mirror pool enable vms pool
        rbd mirror pool enable volumes pool
        ```
    *   添加 peer

        \[openstack]

        ```bash
        rbd mirror pool peer add images client.admin@backup
        rbd mirror pool peer add vms client.admin@backup
        rbd mirror pool peer add volumes client.admin@backup
        ```
    *   启动`ceph-rbd-mirror`服务

        \[openstack]

        ```bash
        yum install -y rbd-mirror
        systemctl start ceph-rbd-mirror@admin.service
        systemctl enable ceph-rbd-mirror@admin.service
        systemctl status ceph-rbd-mirror@admin.service
        ```
    *   查看同步状态

        \[openstack]

        ```bash
        rbd mirror pool status images
        rbd mirror pool status vms
        rbd mirror pool status volumes
        ```

### MariaDB Galera Cluster 恢复

*   停止服务，以主节点方式启动

    \[backup]

    ```bash
    systemctl stop mariadb.service
    galera_new_cluster
    ```
*   导出数据库并拷贝到 openstack 集群服务器

    \[backup]

    ```bash
    mysqldump --all-databases > ~/all.sql
    scp ~/all.sql root@openstack_ip:~/
    systemctl stop mariadb.service
    ```
*   替换 sql 中的 ceph fsid

    \[openstack]

    ```bash
    # 查看FSID
    ceph fsid
    # 替换FSID，将上条命令的结果，替换到FSID处
    sed -i 's/rbd:\/\/.\{36\}/rbd:\/\/FSID/g' ~/all.sql
    ```
*   清空数据库

    \[openstack]

    ```bash
    systemctl stop mariadb.service
    rm -rf /var/lib/mysql/*
    ```
*   导入 sql

    \[openstack]

    ```sql
    # 启动数据库
    galera_new_cluster
    # 登录数据库
    mysql
    # 导入sql
    source ~/all.sql
    ```
