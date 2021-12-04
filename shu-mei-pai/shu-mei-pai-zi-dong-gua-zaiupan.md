# 树莓派自动挂载 U 盘

> udev 规则不能访问网络和挂载/卸载文件系统,所以需要结合 systemd 服务进行挂载/卸载操作

- 安装依赖软件

  ```bash
  sudo apt-get install exfat-fuse ntfs-3g
  ```

- 创建挂载/卸载脚本

  创建文件`/usr/local/bin/usb-mount.sh`内容如下

  ```properties
  #!/bin/bash

  # This script is called from our systemd unit file to mount or unmount
  # a USB drive.

  usage()
  {
      echo "Usage: $0 {add|remove} device_name (e.g. sdb1)"
      exit 1
  }

  if [[ $# -ne 2 ]]; then
      usage
  fi

  ACTION=$1
  DEVBASE=$2
  DEVICE="/dev/${DEVBASE}"

  # See if this drive is already mounted, and if so where
  MOUNT_POINT=$(/bin/mount | /bin/grep ${DEVICE} | /usr/bin/awk '{ print $3 }')

  do_mount()
  {
      if [[ -n ${MOUNT_POINT} ]]; then
          echo "Warning: ${DEVICE} is already mounted at ${MOUNT_POINT}"
          exit 1
      fi

      # Get info for this drive: $ID_FS_LABEL, $ID_FS_UUID, and $ID_FS_TYPE
      eval $(/sbin/blkid -o udev ${DEVICE})

      MOUNT_POINT="/media/usbhd-${DEVBASE}"

      echo "Mount point: ${MOUNT_POINT}"

      /bin/mkdir -p ${MOUNT_POINT}

      # Global mount options
      OPTS="rw,relatime,users,gid=100,umask=000"

      # File system type specific mount options
      if [[ ${ID_FS_TYPE} == "vfat" || ${ID_FS_TYPE} == "ntfs" ]]; then
          OPTS+=",shortname=mixed,utf8=1,flush"
      fi

      if ! /bin/mount -o ${OPTS} ${DEVICE} ${MOUNT_POINT}; then
          echo "Error mounting ${DEVICE} (status = $?)"
          /bin/rmdir ${MOUNT_POINT}
          exit 1
      fi

      echo "**** Mounted ${DEVICE} at ${MOUNT_POINT} ****"
  }

  do_unmount()
  {
      if [[ -z ${MOUNT_POINT} ]]; then
          echo "Warning: ${DEVICE} is not mounted"
      else
          /bin/umount -l ${DEVICE}
          echo "**** Unmounted ${DEVICE}"
      fi

      # Delete all empty dirs in /media that aren't being used as mount
      # points. This is kind of overkill, but if the drive was unmounted
      # prior to removal we no longer know its mount point, and we don't
      # want to leave it orphaned...
      for f in /media/* ; do
          if [[ -n $(/usr/bin/find "$f" -maxdepth 0 -type d -empty) ]]; then
              if ! /bin/grep -q " $f " /etc/mtab; then
                  echo "**** Removing mount point $f"
                  /bin/rmdir "$f"
              fi
          fi
      done
  }

  case "${ACTION}" in
      add)
          do_mount
          ;;
      remove)
          do_unmount
          ;;
      *)
          usage
          ;;
  esac
  ```

- 添加可执行权限

  ```bash
  chmod +x /usr/local/bin/usb-mount.sh
  ```

- 定义挂载/卸载服务

  创建文件`/etc/systemd/system/usb-mount@.service`,内容如下

  ```bash
  [Unit]
  Description=Mount USB Drive on %i
  [Service]
  Type=oneshot
  RemainAfterExit=true
  ExecStart=/usr/local/bin/usb-mount.sh add %i
  ExecStop=/usr/local/bin/usb-mount.sh remove %i
  ```

- 重新加载 systemd

  ```bash
  systemctl daemon-reload
  ```

- 定义 udev 规则

  创建文件`/etc/udev/rules.d/99-usb-mount.rules`,内容如下

  ```properties
  KERNEL=="sd[a-z][0-9]", SUBSYSTEMS=="usb", ACTION=="add", RUN+="/bin/systemctl start usb-mount@%k.service"

  KERNEL=="sd[a-z][0-9]", SUBSYSTEMS=="usb", ACTION=="remove", RUN+="/bin/systemctl stop usb-mount@%k.service"
  ```

- 重新加载规则

  ```bash
  udevadm control --reload-rules
  ```

- 参考链接

  <https://serverfault.com/questions/766506/automount-usb-drives-with-systemd>
