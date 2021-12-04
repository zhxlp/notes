# 迅雷自动安装 Chrome 插件

## 基本信息

插件路径：`C:\Users\Administrator\AppData\Local\ChromeExtensionCache\xl_ext_chrome.crx`

插件 ID：ncennffkjdiamlpmcbajkmaiiiddgioo

插件版本：3.19

## 安装流程

- 添加注册表

  ```properties
  REG ADD "HKLM\SOFTWARE\Google\chrome\Extensions\ncennffkjdiamlpmcbajkmaiiiddgioo" /v path /t REG_SZ /d "C:\Users\Administrator\AppData\Local\ChromeExtensionCache\xl_ext_chrome.crx"  /f /reg:32
  REG ADD "HKLM\SOFTWARE\Google\chrome\Extensions\ncennffkjdiamlpmcbajkmaiiiddgioo" /v version /t REG_SZ /d "3.19"  /f /reg:32
  ```

> 注：插件安装后，默认没有启用，必须手动启用。插件卸载后，必须删除**C:\Users\Administrator\AppData\Local\Google\Chrome\User Data\Default\Secure Preferences**文件中关于插件 ID 的信息，才会重新安装
