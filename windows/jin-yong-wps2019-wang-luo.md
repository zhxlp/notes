# 禁用 wps2019 网络

禁用 wps2019 网络,以实现无广告

- 创建 bat 脚本`block-wps2019.bat`,内容如下

```properties
@echo off
set work_path="C:\Users\Administrator\AppData\Local\kingsoft\WPS Office"
C:
cd %work_path%
for /R %%s in (*.exe) do (
netsh advfirewall firewall add rule name="wps" dir=out program="%%s" action=block
)
pause
```
