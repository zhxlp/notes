# OpenStack-RBD 导入镜像

## 方法 1

*   使用命令导入假镜像

    ```bash
    openstack image create "Ubuntu18.04" \
    --file ~/cirros-0.4.0-x86_64-disk.img \
    --disk-format raw --container-format bare \
    --property hw_scsi_model=virtio-scsi \
    --property hw_disk_bus=scsi \
    --property hw_qemu_guest_agent=yes \
    --property os_require_quiesce=yes \
    --property os_type=linux \
    --property os_admin_user=root \
    --min-disk 20 \
    --min-ram 2048 \
    --public

    # ................
    # id | ef85323a-c0f1-4452-922c-bdb8592fddde
    # .................
    ```
*   rbd 导入

    ```bash
    # 解除快照保护
    rbd snap unprotect images/ef85323a-c0f1-4452-922c-bdb8592fddde@snap
    # 删除快照
    rbd snap rm images/ef85323a-c0f1-4452-922c-bdb8592fddde@snap
    # 删除镜像
    rbd rm images/ef85323a-c0f1-4452-922c-bdb8592fddde
    # 导入镜像 ubuntu18.04.raw
    rbd import ubuntu16.04.raw images/ef85323a-c0f1-4452-922c-bdb8592fddde
    # 创建快照
    rbd snap create images/ef85323a-c0f1-4452-922c-bdb8592fddde@snap
    # 保护快照
    rbd snap protect images/ef85323a-c0f1-4452-922c-bdb8592fddde@snap
    ```
*   计算镜像`md5`和`sha512`值

    ```bash
    # 计算md5值
    md5sum ubuntu18.04.raw
    # ba3cd24377dde5dfdd58728894004abb    ubuntu18.04.raw

    # 计算 sha512值
    sha512sum ubuntu18.04.raw
    # b795f047a1b10ba0b7c95b43b2a481a59289dc4cf2e49845e60b194a911819d3ada03767bbba4143b44c93fd7f66c96c5a621e28dff51d1196dae64974ce240e   ubuntu18.04.raw
    ```
*   连接数据库，更新数据

    ```sql
    UPDATE `glance`.`images`
    -- 镜像大小
    SET `size` = 21474836480,
    -- 镜像md5值
    `checksum` = '7324c73bee3216b5c7a9d27370eb1774',
    -- sha512 校验
    `os_hash_algo` = 'sha512',
    `os_hash_value` = '5ee4ca048ef161f661cf308654cc7e7bdcb9af6905447c1b098aa3df09a27fd8468ee7573f3e092051422c07e6ee67fb1aab5119172c06d551005b5192562778'
    WHERE `id` = 'ef85323a-c0f1-4452-922c-bdb8592fddde';
    ```

## 方法 2

* 导入脚本

```bash
#!/bin/bash
set -e
IMAGE_ID='20b1c7b8-0be6-4380-9fe8-8551adaf2d22'
IMAGE_NAME='WindowsServer2008R2_Enterprise_x64'
IMAGE_SIZE='42949672960'
MIN_DISK='40'
MIN_RAM='2048'
OS_TYPE='windows'
OS_ADMIN_USER='Administrator'
LOGIN_NAME='Administrator'
LOGIN_PASSWORD='qwe123.'
IMAGE_MD5='6e771ab1c21c499038cc8a6baecf92ba'
IMAGE_SHA512='64ffde031cdf9b9ce7c0b5b99a3c59289e632d74e047641954329f4efeffb6aa9ef204bb9ef5d15f85f475741dec2fc2ddb65cd1275c6d3013cae9891097405a'
IMAGE_URL="rbd://$(grep fsid /etc/ceph/ceph.conf | awk -F '=' 'NR == 1 {print $2}' | sed 's/ //g')/images/$IMAGE_ID/snap"
IMAGE_OWNER=`openstack project show admin -c id -f value`


echo "# 导入镜像 $IMAGE_NAME.raw"
echo "rbd import $IMAGE_NAME.raw images/$IMAGE_ID"
echo "# 创建镜像快照"
echo "rbd snap create images/$IMAGE_ID@snap"
echo "# 保护快照"
echo "rbd snap protect images/$IMAGE_ID@snap"

echo "# 连接数据库,执行SQL语句"
echo "-- 插入镜像数据"
echo "INSERT INTO glance.images(id, name, size, status, created_at, updated_at, deleted, disk_format, container_format, checksum, owner, min_disk, min_ram, visibility, os_hash_algo, os_hash_value) VALUES ('$IMAGE_ID', '$IMAGE_NAME', $IMAGE_SIZE, 'active', now(), now(), 0, 'raw', 'bare', '$IMAGE_MD5', '$IMAGE_OWNER', $MIN_DISK, $MIN_RAM, 'public', 'sha512', '$IMAGE_SHA512');"

echo "-- 插入镜像地址"
echo "INSERT INTO glance.image_locations(image_id, value, created_at, updated_at, deleted, meta_data) VALUES ('$IMAGE_ID', '$IMAGE_URL', now(), now(), 0, '{}');"

echo "-- 插入元数据"
echo "INSERT INTO glance.image_properties(image_id, name, value, created_at, updated_at, deleted) VALUES ('$IMAGE_ID', 'hw_qemu_guest_agent', 'yes', now(), now(), 0);"
echo "INSERT INTO glance.image_properties(image_id, name, value, created_at, updated_at, deleted) VALUES ('$IMAGE_ID', 'os_require_quiesce', 'yes', now(), now(), 0);"
echo "INSERT INTO glance.image_properties(image_id, name, value, created_at, updated_at, deleted) VALUES ('$IMAGE_ID', 'os_type', '$OS_TYPE', now(), now(), 0);"
echo "INSERT INTO glance.image_properties(image_id, name, value, created_at, updated_at, deleted) VALUES ('$IMAGE_ID', 'hw_scsi_model', 'virtio-scsi', now(), now(), 0);"
echo "INSERT INTO glance.image_properties(image_id, name, value, created_at, updated_at, deleted) VALUES ('$IMAGE_ID', 'hw_disk_bus', 'scsi', now(), now(), 0);"
echo "INSERT INTO glance.image_properties(image_id, name, value, created_at, updated_at, deleted) VALUES ('$IMAGE_ID', 'os_admin_user', '$OS_ADMIN_USER', now(), now(), 0);"
echo "INSERT INTO glance.image_properties(image_id, name, value, created_at, updated_at, deleted) VALUES ('$IMAGE_ID', 'login_name', '$LOGIN_NAME', now(), now(), 0);"
echo "INSERT INTO glance.image_properties(image_id, name, value, created_at, updated_at, deleted) VALUES ('$IMAGE_ID', 'login_password', '$LOGIN_PASSWORD', now(), now(), 0);"
```
