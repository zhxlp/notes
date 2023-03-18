# Android安卓CA至系统

## 前置条件

具有root权限并且读写挂载system

[android-du-xie-gua-zai-system.md](android-du-xie-gua-zai-system.md "mention")

## 证书转码

Android 证书要求 `pem` 格式,并在文件命名为 `hash值.0`

```
# 转码为 pem 格式
openssl x509 -inform DER -in  FiddlerRoot.cer -out FiddlerRoot.pem
# 查看证书 hash
openssl x509 -inform PEM -subject_hash_old -in FiddlerRoot.pem
# 269953fb
# 重命名
cp FiddlerRoot.pem 269953fb.0
```



## 上传证书

```
adb push 269953fb.0 /system/etc/security/cacerts/
adb reboot
```
