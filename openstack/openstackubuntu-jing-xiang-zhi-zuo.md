# OpenStack Ubuntu 镜像制作

- 创建虚拟机

  ```bash
  virt-install --name Ubuntu16.04_x86 \
  --vcpus 1 \
  --memory 2048 \
  --disk path=/var/lib/libvirt/images/ubuntu16.04_x86.qcow2,size=20,format=qcow2,bus=scsi \
  --disk path=/data/iso/ubuntu-16.04.6-server-i386.iso,device=cdrom,readonly=on \
  --channel unix,target_type=virtio,name=org.qemu.guest_agent.0 \
  --controller scsi,model=virtio-scsi \
  --network bridge=br0,model=virtio \
  --graphics vnc,listen=0.0.0.0 \
  --noautoconsole

  # 查看vnc端口
   virsh vncdisplay Ubuntu16.04_x86
  ```

- 自定义安装操作系统,安装操作系统时使用`ubuntu`用户

- 自定义安装软包和服务

- 安装软件包

  ```bash
  sudo apt install cloud-init cloud-utils qemu-guest-agent openssh-server
  ```

- 配置 openstack 获取日志

  ```bash
  # 在/etc/default/grub文件中设置GRUB_CMDLINE_LINUX_DEFAULT="console=tty0 console=ttyS0,115200n8"
  sudo vi /etc/default/grub
  # 更新grub
  sudo update-grub
  ```

- 配置 cloud-init

  ```bash
  # 编辑/etc/cloud/cloud.cfg,添加或修改如下内容
  disable_root: false
  ssh_pwauth: true

  # 删除nocloud
  sudo rm -rf /var/lib/cloud/seed/*
  ```

- 配置 ssh

  ```bash
  # 编辑/etc/ssh/sshd_config 文件,修改 PermitRootLogin yes , PasswordAuthentication yes
  sudo vi /etc/ssh/sshd_config
  # 重启和设置开机自启
  sudo systemctl restart sshd.service
  sudo systemctl enable sshd.service
  ```

- 设置 root 密码

  ```bash
  sudo passwd root
  ```

- 清理 apt 缓存

  ```bash
  sudo apt clean
  ```

- 关机

  ```bash
  shutdown -h now
  ```

- 转换镜像格式为`raw`

  ```bash
  qemu-img convert -O raw Ubuntu16.04_x86.qcow2 ubuntu16.04_x86.raw
  ```
