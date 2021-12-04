# DOS 脚本

## 向 hosts 文件添加本地回环记录

向 hosts 文件添加 `127.0.0.1 localhost` 和 `::1 localhost`两条记录，并保证不重复添加

```bash
@echo off
:host1
set sHost=127.0.0.1
set sDomain=localhost
FOR /F "eol=# tokens=1,2 delims= " %%i in (C:\Windows\system32\drivers\etc\hosts) do if "%%i"=="%sHost%" if "%%j"=="%sDomain%" goto host2
@ECHO %sHost% %sDomain% >>C:\Windows\System32\drivers\etc\hosts

:host2
set sHost=::1
set sDomain=localhost
FOR /F "eol=# tokens=1,2 delims= " %%i in (C:\Windows\system32\drivers\etc\hosts) do if "%%i"=="%sHost%" if "%%j"=="%sDomain%" goto end
@ECHO %sHost% %sDomain% >>C:\Windows\System32\drivers\etc\hosts

:end
ipconfig /flushdns
@ECHO  "按任意键退出"
@pause
@exit
```

## 在文件管理器添加派盘

```bash
# 添加
REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4} /t REG_SZ /d 派盘 /f
REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4} /v InfoTip /t REG_SZ /d 从这里进入派盘 /f
REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4} /v TileInfo /t REG_SZ /d prop:System.ItemAuthors /f
REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4} /v System.ItemAuthors /t REG_SZ /d 双击进入派盘 /f
REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4} /v System.IsPinnedToNamespaceTree /t REG_DWORD /d 1 /f
REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4} /v SortOrderIndex /t REG_DWORD /d 66 /f

REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4}\DefaultIcon /t REG_EXPAND_SZ /d "C:\Program Files (x86)\HULUER\iChainPi\logo.ico" /f

REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4}\InProcServer32 /t REG_EXPAND_SZ /d ^%SystemRoot^%\system32\shdocvw.dll /f
REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4}\InProcServer32 /v ThreadingModel /t REG_SZ /d Apartment /f

REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4}\Instance /v CLSID /t REG_SZ /d {0AFACED1-E828-11D1-9187-B532F1E9575D} /f

REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4}\Instance\InitPropertyBag /v Attributes /t REG_DWORD /d 17 /f
REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4}\Instance\InitPropertyBag /v Target /t REG_SZ /d C:\iChainPi\ /f

REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4}\ShellFolder /v FolderValueFlags /t REG_DWORD /d 40 /f
REG ADD HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4}\ShellFolder /v Attributes /t REG_DWORD /d 4034920525 /f

REG ADD HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Desktop\NameSpace\{85199E68-7F10-4E93-BD56-1499D50A08F4} /t REG_SZ /d 派盘 /f

REG ADD HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\HideDesktopIcons\NewStartPanel /v {85199E68-7F10-4E93-BD56-1499D50A08F4} /t REG_DWORD /d 1 /f

REG ADD HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{85199E68-7F10-4E93-BD56-1499D50A08F4} /t REG_SZ /d 派盘 /f



# 删除
REG DELETE HKCU\SOFTWARE\Classes\CLSID\{85199E68-7F10-4E93-BD56-1499D50A08F4} /f

REG DELETE HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Desktop\NameSpace\{85199E68-7F10-4E93-BD56-1499D50A08F4} /f

REG DELETE HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{85199E68-7F10-4E93-BD56-1499D50A08F4} /f

REG DELETE HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\HideDesktopIcons\NewStartPanel /v {85199E68-7F10-4E93-BD56-1499D50A08F4} /f
```
