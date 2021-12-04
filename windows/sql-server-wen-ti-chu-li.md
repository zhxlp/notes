# SQL Server 问题处理

## 等待数据库引擎恢复句柄失败

### 问题

数据库安装时错误，提示如下

![](../.gitbook/assets/Snipaste_2019-09-29_01-40-25.png)

### 原因

数据库引擎服务权限不足导致

### 方法一

卸载重新安装数据库，在【服务器配置】时，【服务账号】中【SQL Server 数据库引擎】选择【NT AUTHORITY\SYSTEM】用户。

### 方法二

继续安装过程，安装完成后，打开【服务】面板，【SqlServer】服务设置本地系统用户启动。在安装文件的 Setup.exe 文件目录打开命令行,输入如下内容进行重新生成系统数据库

```bash
Setup /QUIET /ACTION=REBUILDDATABASE /instancename=MSSQLSERVER /SQLSYSADMINACCOUNTS=BUILTIN\Administrators  /sapwd=abc*123 /sqlcollation=Chinese_PRC_CI_AS
```

## 参考文档

- https://blog.csdn.net/weixin_30649641/article/details/99284809
- https://blog.csdn.net/chxljtt/article/details/2032150
- https://docs.microsoft.com/zh-cn/sql/relational-databases/databases/rebuild-system-databases?view=sql-server-2014
- https://docs.microsoft.com/zh-cn/sql/relational-databases/collations/set-or-change-the-server-collation?view=sql-server-2017
-
