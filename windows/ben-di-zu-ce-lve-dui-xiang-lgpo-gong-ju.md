# 本地组策略对象 (LGPO) 工具

## 简介

LGPO.exe 是旨在帮助自动化管理本地群组策略的命令行实用工具。 使用本地策略为管理员提供了一种简单的方法来验证群组策略设置的效果，并且对于管理非域加入的系统也很有用。 LGPO.exe 可以导入并应用注册表策略 (Registry.pol) 文件、安全模板、高级审计备份文件以及格式化的“LGPO 文本”文件中的设置。 它可以将本地策略导出到 GPO 备份。 它可以将注册表策略文件的内容导出为可以编辑的“LGPO 文本”格式，并可以根据 LGPO 文本文件构建注册表策略文件。

LGPO 工具的文档可在[Microsoft 安全指南博客](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-compliance-toolkit-10)或[下载工具](https://www.microsoft.com/download/details.aspx?id=55319)中找到。

> 相关工具：PolicyPlus

## 备份/还原本地策略组

```properties
# 备份本地策略组
LGPO.exe /b path [/n GPO-name]
path 导出路径(全路径)
# 还原本地策略组
LGPO.exe /g path
```

## 将 Registry.pol 解析为 LGPO text

```properties
LGPO.exe /parse [/q] {/m|/u|/ua|/un|/u:username} path\registry.pol
/q 安静输出
/m 计算机配置
/u 系统范围的用户配置
/ua 管理员用户配置
/un 非管理员用户配置
/u:username 指定用户配置
```

## 从 LGPO text 构建 Registry.pol

```properties
 LGPO.exe /r path\lgpo.txt /w path\registry.pol [/v]
 /r path\lgpo.txt 读取LGPO文本文件
 /w path\registry.pol 写入新的 registry.pol 文件
 /v 详细输出
```

## 从 pol 导入 Registry 设置

```properties
LGPO.exe {/m|/u|/ua|/un|/u:username} path\registry.pol
/m 计算机配置
/u 系统范围的用户配置
/ua 管理员用户配置
/un 非管理员用户配置
/u:username 指定用户配置
```

## 从 LGPO text 导入 Registry 设置

```properties
LGPO.exe /t path\lgpo.txt
```

## 导入 Security 模板

```properties
LGPO.exe /s path\GptTmpl.inf
```
