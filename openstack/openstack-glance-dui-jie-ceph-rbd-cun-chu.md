# OpenStack Glance 对接 Ceph RBD 存储

## 创建存储池

默认情况下， Ceph 块设备使用 `rbd` 存储池。你可以用任何可用存储池。建议为 Cinder 和 Glance 单独创建池。确保 Ceph 集群在运行，然后创建存储池。

```bash
ceph osd pool create images 64
rbd pool init images
```

## 配置 OpenStack 的 Ceph 客户端

运行着 `glance-api`的主机被当作 Ceph 客户端，它们都需要 `ceph.conf` 文件。

```bash
ssh {your-openstack-server} sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
```

### 安装 CEPH 客户端软件包

在运行 `glance-api` 的节点上你需要 `librbd` 的 Python 绑定：

```bash
yum install python-rbd
```

### 配置 CEPH 客户端认证

如果你启用了 [cephx 认证](http://docs.ceph.org.cn/rados/operations/authentication)，需要创建 Glance 用户：

```bash
ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images'
```

把 `client.glance` 的密钥环复制到适当的节点，并更改所有权：

```bash
ceph auth get-or-create client.glance | ssh {your-glance-api-server} sudo tee /etc/ceph/ceph.client.glance.keyring
ssh {your-glance-api-server} sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring
```

## 配置 OPENSTACK 使用 CEPH

Glance 可使用多种后端存储 image 。要让它默认使用 Ceph 块设备，按如下配置 Glance 。

编辑 `/etc/glance/glance-api.conf` 并把下列内容加到 `[glance_store]` 段下：

```bash
[glance_store]
default_store = rbd
stores = rbd
rbd_store_pool = images
rbd_store_user = glance
rbd_store_ceph_conf = /etc/ceph/ceph.conf
rbd_store_chunk_size = 8
```

如果你想允许使用 image 的写时复制克隆，再添加下列内容到 `[DEFAULT]` 段下：

```bash
show_image_direct_url = True
```

注意，这会通过 Glance API 暴露后端存储位置，所以此选项启用时 endpoint 不应该被公开访问。

禁用 Glance 缓存管理，以免 image 被缓存到 `/var/lib/glance/image-cache/` 下，假设你的配置文件里有 `flavor = keystone+cachemanagement` ：

```bash
[paste_deploy]
flavor = keystone
```

## 重启服务

`glance-api`节点

```bash
systemctl restart openstack-glance-api.service
```

## IMAGE 属性

建议配置如下 image 属性：

- `hw_scsi_model=virtio-scsi`: 添加 virtio-scsi 控制器以获得更好的性能、并支持 discard 操作；
- `hw_disk_bus=scsi`: 把所有 cinder 块设备都连到这个控制器；
- `hw_qemu_guest_agent=yes`: 启用 QEMU guest agent （访客代理）
- `os_require_quiesce=yes`: 通过 QEMU guest agent 发送 fs-freeze/thaw 调用
