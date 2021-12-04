# CentOS7 KVM GPU 穿透

- 在 BIOS 中开启 VT-d 和 VT-x。

- 检查 cpu 开启虚拟化

  ```bash
  egrep -o '(vmx|svm)' /proc/cpuinfo
  ```

- 内核开启 iommu

  ```bash
  # 编辑grub文件，在“GRUB_CMDLINE_LINUX”行添加“intel_iommu=on"
  vi /etc/default/grub
  # 更新grub文件
  grub2-mkconfig -o /boot/grub2/grub.cfg
  # 重启系统
  reboot
  # 检查是否开启，结果中查看是否有"IOMMU enabled"
  dmesg | grep -e DMAR -e IOMMU
  ```

- 从宿主机分离 NVIDIA 设备

  ```bash
  dmesg | grep NVIDIA
  #[    3.378181] nouveau 0000:06:00.0: NVIDIA GM204 (124320a1)
  # 查看设备列表中是否存在
  virsh nodedev-list | grep pci_0000_06_00_0
  # 获取设备xml
  virsh nodedev-dumpxml pci_0000_06_00_0 | grep NVIDIA
  # 分离设备
  virsh nodedev-dettach pci_0000_06_00_0
  ```

- 在 kvm 虚拟机添加 NVIDIA 设备
