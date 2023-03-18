# Android读写挂载system

## 前置条件

具有root权限的设备或虚拟机，例如：

[android-studio-chuang-jian-ju-you-root-quan-xian-de-xu-ni-ji.md](android-studio-chuang-jian-ju-you-root-quan-xian-de-xu-ni-ji.md "mention")



## 步骤

启动虚拟机

```
emulator -writable-system -netdelay none -netspeed full -avd Pixel_5_API_31
```

读写挂载

```
# 查看设备
adb devices
# 终端root
adb root
# 禁用系统验证
adb disable-verity
# 重启
adb reboot
# 终端root
adb root
# 读写挂载 system
adb remount
```
