# rbd-mirror

- 参考链接

  https://ceph.com/planet/rbd-mirror配置指南-单向备份/
  https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3/html/block_device_guide/block_device_mirroring
  https://docs.ceph.com/docs/mimic/rbd/rbd-mirroring/

- 环境

  Ceph：Luminous

  pool：images

  正常部署两套 ceph 集群，为了好区分，一套代号未 master，一套代号为 slave。

  目标：salve 同步 master 的数据。

- 创建具有日志功能的块或者启用块的日志功能

  RBD 需要启用`exclusive-lock`和`journaling`属性,可以手动创建时启用，也可以修改配置文件，创建时自动启用。

  [master]

  ```bash
  # 创建时启用日志功能
  # rbd create <pool-name>/<image-name> --size <megabytes> --image-feature <feature>
  rbd create images/image1 --size 1024 --image-feature exclusive-lock,journaling

  # 创建后启用日志功能
  # rbd feature enable <pool-name>/<image-name> <feature-name>
  rbd feature enable images/image1 exclusive-lock
  rbd feature enable images/image1 journaling

  # 修改配置文件,开启自动启动功能
  crudini --set /etc/ceph/ceph.conf 'global' 'rbd default features' '125'
  systemctl restart ceph.target
  ```

- 开启 mirror 模式

  mirror 有 pool 和 image 两张模式

  pool 模式：启用日志功能的 rbd 都开启 mirror

  image 模式：指定的 rbd 开启 mirror

  [all]

  ```bash
  # 启用mirror
  # rbd mirror pool enable <pool-name> <mode>
  rbd mirror pool enable images image

  # 禁用mirror
  # rbd mirror pool disable <pool-name>
  ```

- 拷贝配置文件和密钥到 salve

  将 master 的配置文件和密钥拷贝到 slave

  [master]

  ```bash
  scp /etc/ceph/ceph.conf root@slave:/etc/ceph/master.conf
  scp /etc/ceph/ceph.client.admin.keyring root@slave:/etc/ceph/master.client.admin.keyring
  ```

- 增加 peer

  [slave]

  ```bash
  # rbd mirror pool peer add <pool-name> <client-name>@<cluster-name>
  rbd mirror pool peer add images client.admin@master

  # 查看peer状态
  # rbd mirror pool info <pool-name>
  rbd mirror pool info images
  ```

- 启动`ceph-rbd-mirror`

  [slave]

  ```bash
  yum install -y rbd-mirror
  systemctl start ceph-rbd-mirror@admin.service
  ```

- 查看 rbd 状态

  [slave]

  ```bash
  # rbd mirror pool status <pool-name>
  rbd mirror pool status images
  ```
