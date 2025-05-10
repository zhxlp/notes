# 禁用应用联网



禁用应用联网脚本

按照应用不同修改`block_rule_name`与 `path_list`

```
@echo off
setlocal enabledelayedexpansion


:: 定义阻拦规则名称(应用唯一)
set block_rule_name="Adobe-Block"

:: 定义路径列表
set path_list="C:\Program Files\Adobe" "C:\Program Files (x86)\Adobe" "C:\Program Files\Common Files\Adobe" "C:\Program Files (x86)\Common Files\Adobe"

:: 删除旧规则
netsh advfirewall firewall delete rule name="%block_rule_name%"


:: 遍历列表中的每个路径
for %%p in (%path_list%) do (
    set "current_path=%%p"
    echo 正在处理路径: !current_path!
    
    if exist "!current_path!\" (
        pushd "!current_path!"
        for /R %%s in (*.exe) do (
            echo 正在为 %%s 创建防火墙规则...
            netsh advfirewall firewall add rule name="%block_rule_name%" dir=out program="%%s" action=block
        )
        popd
    ) else (
        echo 路径不存在: !current_path!
    )
)

echo 所有路径处理完成
pause
```



禁用 wps2019 网络

```
:: 定义阻拦规则名称(应用唯一)
set block_rule_name="WPS-Office-Block"

:: 定义路径列表
set path_list="C:\Users\Administrator\AppData\Local\kingsoft\WPS Office"
```



