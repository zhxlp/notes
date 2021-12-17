# 华为一碰传标签制作

```bash
# 禁用app
adb shell pm disable-user com.huawei.android.instantshare
adb shell pm disable-user com.huawei.pcassistant
adb shell pm disable-user com.huawei.iconnect
adb shell pm disable-user com.android.nfc
adb shell pm disable-user com.huawei.android.internal.app

# 清理app存储
adb shell pm clear com.huawei.android.instantshare
adb shell pm clear com.huawei.pcassistant
adb shell pm clear com.huawei.iconnect
adb shell pm clear com.android.nfc
adb shell pm clear com.huawei.android.internal.app

# 重启手机
adb shell reboot

# 查看已禁用
adb shell pm list packages -d


# 启用nfs app
adb shell pm enable com.android.nfc

# 使用 NFC Tool 清理 nfs 标签
# 使用 一碰传助手 制作nfc标签

# 启用app
adb shell pm enable com.huawei.android.instantshare
adb shell pm enable com.huawei.pcassistant
adb shell pm enable com.huawei.iconnect
adb shell pm enable com.android.nfc
adb shell pm enable com.huawei.android.internal.app

# 重启手机
adb shell reboot

```
