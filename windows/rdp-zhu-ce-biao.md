# RDP 注册表

## 设置远程 DLL 文件路径

```bash
REG ADD "HKLM\SYSTEM\CurrentControlSet\Services\TermService\Parameters" /v "ServiceDll" /t REG_EXPAND_SZ /d "%SystemRoot%\System32\termsrv.dll" /f
```

## 启动远程桌面

```bash
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v "fDenyTSConnections" /t REG_DWORD /d 0 /f
```

## 设置端口

```bash
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "PortNumber" /t REG_DWORD /d 3389 /f
# 设置防火墙
netsh advfirewall firewall set rule name="Remote Desktop" new localport=3389
```

## 每用户单会话

```bash
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v "fSingleSessionPerUser" /t REG_DWORD /d 1 /f
```

## 允许启动自定义程序

```bash
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v "HonorLegacySettings" /t REG_DWORD /d 1 /f
```

## 身份验证模式

### GUI Authentication

```bash
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "SecurityLayer" /t REG_DWORD /d 0 /f
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "UserAuthentication" /t REG_DWORD /d 0 /f
```

### Default RDP Authentication

```bash
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "SecurityLayer" /t REG_DWORD /d 1 /f
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "UserAuthentication" /t REG_DWORD /d 0 /f
```

### Network Level Authentication

```bash
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "SecurityLayer" /t REG_DWORD /d 2 /f
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "UserAuthentication" /t REG_DWORD /d 1 /f
```

## 远程协助模式

```bash
# 禁用
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "Shadow" /t REG_DWORD /d 0 /f
REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v "Shadow" /t REG_DWORD /d 0 /f

# 在用户许可下完全访问
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "Shadow" /t REG_DWORD /d 1 /f
REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v "Shadow" /t REG_DWORD /d 1 /f

# 未经许可完全访问
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "Shadow" /t REG_DWORD /d 2 /f
REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v "Shadow" /t REG_DWORD /d 2 /f

# 仅在用户许可下查看
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "Shadow" /t REG_DWORD /d 3 /f
REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v "Shadow" /t REG_DWORD /d 3 /f

# 未经许可的情况下查看
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v "Shadow" /t REG_DWORD /d 4 /f
REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v "Shadow" /t REG_DWORD /d 4 /f
```

## 显示上次登录的用户名

```bash
REG ADD "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v "DontDisplayLastUserName" /t REG_DWORD /d 0 /f
```
