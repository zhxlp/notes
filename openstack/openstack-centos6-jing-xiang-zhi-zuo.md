# OpenStack CentOS6 镜像制作

- 创建虚拟机

  ```bash
  virt-install --name CentOS6.5_x86 \
  --vcpus 1 \
  --memory 2048 \
  --disk path=/var/lib/libvirt/images/CentOS6.5_x86.qcow2,size=20,format=qcow2,bus=scsi \
  --disk path=/data/iso/CentOS-6.5-i386-minimal.iso,device=cdrom,readonly=on \
  --channel unix,target_type=virtio,name=org.qemu.guest_agent.0 \
  --controller scsi,model=virtio-scsi \
  --network bridge=br0,model=virtio \
  --graphics vnc,listen=0.0.0.0 \
  --noautoconsole

  ```

- 配置网络

  ```bash
  echo '
  DEVICE=eth0
  TYPE=Ethernet
  ONBOOT=yes
  NM_CONTROLLED=yes
  BOOTPROTO=dhcp
  ' > /etc/sysconfig/network-scripts/ifcfg-eth0
  # 重启网络服务
  service network restart
  ```

- 配置 sshd

  ```bash
  # 编辑/etc/ssh/sshd_config 文件,修改 PermitRootLogin yes , PasswordAuthentication yes
  sudo vi /etc/ssh/sshd_config
  # 重启sshd
  service restart sshd
  ```

- 配置电源管理

  ```bash
  yum install -y acpid
  ```

- 配置 qemu 代理

  ```bash
  yum install -y qemu-guest-agent
  ```

- 配置 Console

  ```bash
  echo '
  s0:2345:respawn:/sbin/agetty -L -f /etc/issue.serial 115200 ttyS0 vt100
  ' >> /etc/inittab

  echo '
  ttyS0
  ' >> /etc/securetty

  # 修改 /boot/grub/grub.conf文件，在kernel行最后加console=tty0 console=ttyS0,115200n8
  ```

- 禁用 ZEROCONF

  ```bash
  echo "NOZEROCONF=yes" >> /etc/sysconfig/network
  ```

- 配置 cloud-init

  ```bash
  yum install -y cloud-init
  # 编辑 /etc/cloud/cloud.cfg
  # 设置 disable_root: 0
  # 设置 ssh_pwauth:   1
  ```

- 清理关机

  ```bash
  yum clean all
  history -c
  poweroff
  ```

- 清理镜像

  ```bash
  virt-sysprep -d CentOS6.5_x86
  ```

- 转换镜像格式

  ```bash
  qemu-img convert -O raw CentOS6.5_x86.qcow2 CentOS6.5_x86.raw
  ```
