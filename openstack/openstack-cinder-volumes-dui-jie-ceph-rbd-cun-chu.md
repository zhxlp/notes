# OpenStack Cinder Volumes 对接 Ceph RBD 存储

## 创建存储池

默认情况下， Ceph 块设备使用 `rbd` 存储池。你可以用任何可用存储池。建议为 Cinder 单独创建池。确保 Ceph 集群在运行，然后创建存储池。

```bash
ceph osd pool create volumes 64
ceph osd pool create vms 64
rbd pool init volumes
rbd pool init vms
```

## 配置 OPENSTACK 的 CEPH 客户端

运行着 `cinder-volume` 的主机被当作 Ceph 客户端，它们都需要 `ceph.conf` 文件。

```bash
ssh {your-openstack-server} sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
```

### 安装 CEPH 客户端软件包

在 `cinder-volume` 节点上，要安装 Python 绑定和客户端命令行工具：

```bash
yum install ceph
```

### 配置 CEPH 客户端认证

如果你启用了 [cephx 认证](http://docs.ceph.org.cn/rados/operations/authentication)，需要分别为 Cinder 创建新用户。命令如下：

```bash
ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd-read-only pool=images'
```

把 `client.cinder` 的密钥环复制到适当的节点，并更改所有权：

```bash
ceph auth get-or-create client.cinder | ssh {your-volume-server} sudo tee /etc/ceph/ceph.client.cinder.keyring
ssh {your-cinder-volume-server} sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring
```

运行 `nova-compute` 的节点，其进程需要密钥环文件：

```bash
ceph auth get-or-create client.cinder | ssh {your-nova-compute-server} sudo tee /etc/ceph/ceph.client.cinder.keyring
```

还得把 `client.cinder` 用户的密钥存进 `libvirt` 。 libvirt 进程从 Cinder 挂载块设备时要用它访问集群。

在运行 `nova-compute` 的节点上创建一个密钥的临时副本：

```bash
ceph auth get-key client.cinder | ssh {your-compute-node} tee client.cinder.key
```

然后，在**计算节点**上把密钥加进 `libvirt` 、然后删除临时副本：

```bash
uuidgen
#457eb676-33da-42ec-9a8c-9293d545c337

cat > secret.xml <<EOF
<secret ephemeral='no' private='no'>
  <uuid>457eb676-33da-42ec-9a8c-9293d545c337</uuid>
  <usage type='ceph'>
        <name>client.cinder secret</name>
  </usage>
</secret>
EOF

sudo virsh secret-define --file secret.xml
#Secret 457eb676-33da-42ec-9a8c-9293d545c337 created

sudo virsh secret-set-value --secret 457eb676-33da-42ec-9a8c-9293d545c337 --base64 $(cat client.cinder.key) && rm client.cinder.key secret.xml
```

## 配置 OPENSTACK 使用 CEPH

OpenStack 需要一个驱动和 Ceph 块设备交互。还得指定块设备所在的存储池名。编辑 `cinder-volume` 节点上的 `/etc/cinder/cinder.conf` ，添加：

```properties
[DEFAULT]
...
enabled_backends = ceph
...
[ceph]
volume_driver = cinder.volume.drivers.rbd.RBDDriver
rbd_pool = volumes
rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_flatten_volume_from_snapshot = false
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = -1
glance_api_version = 2
```

注意如果你为 cinder 配置了多后端， `[DEFAULT]` 节中必须有 `glance_api_version = 2` 。

如果你使用了 [cephx 认证](http://docs.ceph.org.cn/rados/operations/authentication)，还需要配置用户及其密钥（前述文档中存进了 `libvirt` ）的 uuid ：

```properties
[ceph]
...
rbd_user = cinder
rbd_secret_uuid = 457eb676-33da-42ec-9a8c-9293d545c337
```

## 重启服务

**计算**节点

```bash
systemctl restart openstack-nova-compute.service
```

`cinder-volume` 节点

```bash
systemctl restart openstack-cinder-volume.service
```
