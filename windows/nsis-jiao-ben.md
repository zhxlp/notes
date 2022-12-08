# NSIS 脚本

## 设置文件属性

```properties
; 自定义常量
!define PRODUCT_NAME "zhxlp" ; 程序名称
!define PRODUCT_VERSION "1.0.0.0"  ; 版本

; 安装程序包含的语言
!insertmacro MUI_LANGUAGE "SimpChinese"

; 设置文件属性
VIProductVersion "1.0.0.0" ; 版本号
VIAddVersionKey  /LANG=${LANG_SimpChinese} "ProductName" "${PRODUCT_NAME}" ;产品名称
VIAddVersionKey  /LANG=${LANG_SimpChinese} "Comments" "测试" ;备注
VIAddVersionKey  /LANG=${LANG_SimpChinese} "CompanyName" "zhxlp" ;公司名称
VIAddVersionKey  /LANG=${LANG_SimpChinese} "LegalCopyright" "Copyright (c) 2016-2020 zhxlp" ;版权
VIAddVersionKey  /LANG=${LANG_SimpChinese} "FileDescription" "测试" ;文件描述
VIAddVersionKey  /LANG=${LANG_SimpChinese} "FileVersion" "${PRODUCT_VERSION}" ;文件版本
VIAddVersionKey  /LANG=${LANG_SimpChinese} "ProductVersion" "${PRODUCT_VERSION}" ;产品版本
```

## 单文件可执行文件

```properties
!include  "MUI.nsh"

; 自定义常量
!define PRODUCT_NAME "zhxlp" ; 程序名称
!define PRODUCT_VERSION "1.0.0.0"  ; 版本
!define MUI_ICON icon.ico ; 程序图标

; 安装目录
InstallDir "$TEMP\zhxlp"

Name ${PRODUCT_NAME}
; 输出文件
OutFile "${PRODUCT_NAME}-${PRODUCT_VERSION}.exe"
; 静默安装
SilentInstall silent

Section MainSection
  ; 设置输出目录
  SetOutPath "$INSTDIR"
  ; 文件覆盖
  SetOverwrite on
  ; 释放文件
  File /r "file\*.*"
  ExecShell "open" "$INSTDIR\zhxlp.exe"
SectionEnd
```

## 设置卸载信息

```properties
; 自定义常量
!define PRODUCT_NAME "zhxlp" ;产品名称
!define PRODUCT_VERSION "1.0.0.0" ;版本号
!define COMPANY_NAME "zhxlp" ;公司名称
!define PRODUCT_PUBLISHER "${COMPANY_NAME}" ;发布者,在控制面板-程序和功能中显示
!define PRODUCT_UNINST_KEY "Software\Microsoft\Windows\CurrentVersion\Uninstall\zhxlp" ; 卸载注册表存放位置
!define PRODUCT_WEB_SITE "https://www.zhxlp.com" ; 应用程序主页链接,在控制面板-程序和功能中显示


; 安装完成后,在注册表中写入卸载信息
Section -Post
    ; 软件版本
    WriteUninstaller "$INSTDIR\uninst.exe"
    ; 程序名称
    WriteRegStr HKLM "${PRODUCT_UNINST_KEY}" "DisplayName" "${PRODUCT_NAME}"
    ; 卸载程序路径
    WriteRegStr HKLM "${PRODUCT_UNINST_KEY}" "UninstallString" "$INSTDIR\uninst.exe"
    ; 图标路径
    WriteRegStr HKLM "${PRODUCT_UNINST_KEY}" "DisplayIcon" "$INSTDIR\uninst.exe"
    ; 程序版本
    WriteRegStr HKLM "${PRODUCT_UNINST_KEY}" "DisplayVersion" "${PRODUCT_VERSION}"
    ; 应用程序主页
    WriteRegStr HKLM "${PRODUCT_UNINST_KEY}" "URLInfoAbout" "${PRODUCT_WEB_SITE}"
    ; 发布者名称
    WriteRegStr HKLM "${PRODUCT_UNINST_KEY}" "Publisher" "${PRODUCT_PUBLISHER}"
    ; 安装内容总大小
    ${GetSize} "$INSTDIR" "/S=0K" $0 $1 $2
    IntFmt $0 "0x%08X" $0
    WriteRegDWORD HKLM "${PRODUCT_UNINST_KEY}" "EstimatedSize" "$0"
SectionEnd
Section Uninstall
    ; 删除卸载信息注册表
    DeleteRegKey HKLM "${PRODUCT_UNINST_KEY}"
SectionEnd
```

## 界面内容设置

```properties
!include "MUI.nsh"
; 定义变量
Var MyMsgBeforeRemove ;是否卸载弹框提示内容
var Wtitle ; 欢迎界面标题
var Wtext ; 欢迎界面内容
;var Ftitle ; 完成界面标题
;var Ftext ; 完成界面内容
Var DataDir ; 数据存储目录

; MUI 预定义常量
!define MUI_ABORTWARNING ;定义停止安装警告弹窗
!define MUI_ICON "icon\zhxlp.ico"  ; 安装程序icon图标
!define MUI_WELCOMEFINISHPAGE_BITMAP "icon\install_left.bmp" ; 安装界面左侧图片
!define MUI_TEXT_WELCOME_INFO_TITLE $Wtitle ; 安装欢迎界面标题
!define MUI_TEXT_WELCOME_INFO_TEXT $Wtext ; 安装欢迎界面内容
;!define MUI_TEXT_FINISH_INFO_TITLE $Ftitle ; 安装完成界面标题
;!define MUI_TEXT_FINISH_INFO_TEXT $Ftext ; 安装完成界面内容
!define MUI_UNICON "icon\zhxlp.ico" ; 卸载程序icon图标
;!define MUI_UNWELCOMEFINISHPAGE_BITMAP "icon\install_left.bmp" ; 卸载界面左侧图片
;!define MUI_UNTEXT_WELCOME_INFO_TITLE $Wtitle ; 卸载欢迎界面标题
;!define MUI_UNTEXT_WELCOME_INFO_TEXT $Wtext ; 卸载欢迎界面内容
;!define MUI_UNTEXT_FINISH_INFO_TITLE $Ftitle ; 卸载完成界面标题
;!define MUI_UNTEXT_FINISH_INFO_TEXT $Ftext ; 卸载完成界面内容


; 定义展示界面
!insertmacro MUI_PAGE_WELCOME  ; 安装欢迎页面
;!insertmacro MUI_PAGE_DIRECTORY  ; 安装目录选择页面
!insertmacro MUI_PAGE_INSTFILES ; 安装过程页面
;!insertmacro MUI_PAGE_FINISH ; 安装完成页面
;!insertmacro MUI_UNPAGE_WELCOME ; 卸载欢迎页面
!insertmacro MUI_UNPAGE_INSTFILES ; 卸载过程页面
;!insertmacro MUI_UNPAGE_FINISH ; 卸载完成界面


; 安装程序即将完成初始化时调用
Function .onInit
    ;StrCpy $Wtitle "安装欢迎界面标题"
    ;StrCpy $Wtext "安装欢迎界面内容"
    ;StrCpy $Ftitle "安装完成界面标题"
    ;StrCpy $Ftext "安装完成界面内容"
FunctionEnd

; 卸载程序即将完成初始化时调用
Function un.onInit
    ;StrCpy $Wtitle "卸载欢迎界面标题"
    ;StrCpy $Wtext "卸载欢迎界面内容"
    ;StrCpy $Ftitle "卸载完成界面标题"
    ;StrCpy $Ftext "卸载完成界面内容"
    StrCpy $MyMsgBeforeRemove "你确实要完全卸载 ${PRODUCT_NAME} 其及所有的组件吗？"
    ; 展示是否卸载提示弹窗
    MessageBox MB_ICONQUESTION|MB_YESNO|MB_DEFBUTTON2 $MyMsgBeforeRemove IDYES +2
    Abort   ; 立刻退出
FunctionEnd
```
