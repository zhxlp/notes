# 使用QEMU运行ARM64 CentOS7

```properties
https://gist.github.com/billti/d904fd6124bf6f10ba2c1e3736f0f0f7
https://releases.linaro.org/components/kernel/uefi-linaro/16.02/release/qemu64/QEMU_EFI.fd

# 无法启动
https://blog.csdn.net/WMX843230304WMX/article/details/102628133

# cpu 列表
qemu-system-aarch64.exe -cpu help

# device 列表
qemu-system-aarch64.exe -device help

# 创建镜像
qemu-img.exe create centos_7_5_arm64.img 40G


qemu-system-aarch64.exe ^
-M virt ^
-cpu cortex-a72 ^
-smp 8 ^
-m 4096 ^
-bios QEMU_EFI.fd ^
-device virtio-scsi-device ^
-drive if=none,file=CentOS-7-aarch64-Minimal-1804.iso,id=cdrom,media=cdrom ^
-device scsi-cd,drive=cdrom ^
-drive if=none,file=centos_7_5_arm64.img,id=hd0 ^
-device virtio-blk-device,drive=hd0 ^
-device virtio-net-device,netdev=net0 ^
-netdev user,hostfwd=tcp:127.0.0.1:2222-:22,id=net0 ^
-device nec-usb-xhci -device usb-kbd -device usb-mouse -device VGA




qemu-system-aarch64.exe ^
-M virt ^
-cpu cortex-a72 ^
-smp 8 ^
-m 4096 ^
-bios QEMU_EFI.fd ^
-drive if=none,file=centos_7_5_arm64.img,id=hd0 ^
-device virtio-blk-device,drive=hd0 ^
-device virtio-net-device,netdev=net0 ^
-netdev user,hostfwd=tcp:127.0.0.1:2222-:22,id=net0 ^
-device nec-usb-xhci -device usb-kbd -device usb-mouse -device VGA


qemu-system-aarch64.exe ^
-M virt ^
-cpu cortex-a72 ^
-smp 8 ^
-m 4096 ^
-bios QEMU_EFI.fd ^
-drive if=none,file=centos_7_5_arm64.img,id=hd0 ^
-device virtio-blk-device,drive=hd0 ^
-device virtio-net-device,netdev=net0 ^
-netdev user,hostfwd=tcp:127.0.0.1:2222-:22,id=net0 ^
-nographicc
```
