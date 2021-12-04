# Windows 问题集合

## 清除文件默认程序

```bash
# 清除 .rpm 文件默认程序
REG DELETE "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.rpm" /f
REG DELETE "HKCR\.rpm" /f
```
