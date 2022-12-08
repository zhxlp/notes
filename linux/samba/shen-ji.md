# 审计



详细信息请查看 `man vfs_full_audit`

## 配置



### log level

日志配置

例如： `log level = 1 full_audit:1@/var/log/samba/audit.log`

### vfs objects = full\_audit

启用审计

### full\_audit:prefix

消息前缀

例如：`full_audit:prefix = %S|%u|%I|%m`

日志：张三|zhangsan|192.168.2.231|desktop-g2gfjfi|rmdir|ok|新建文件夹 (2)



### full\_audit:success

需要记录的成功操作列表

例如：`full_audit:success = mkdir rmdir rename unlink`

### full\_audit:failure

需要记录的失败操作列表

例如：`full_audit:failure = connect`

### full\_audit:syslog

将消息记录到syslog（默认）或作为调试级别1消息

例如：`full_audit:syslog = false`

### full\_audit:facility

syslog 设施层级定义

* auth: 身份验证相关的消息（登录时）
* cron: 进程或应用调度相关的消息
* daemon: 守护进程相关的消息（内部服务器）
* kernel: 内核相关的消息
* mail: 内部邮件服务器相关的消息
* syslog: syslog 守护进程本身相关的消息
* lpr: 打印服务相关的消息
* local0 - local7: 用户自定义的消息 （local7 通常被Cisco 和 Windows 服务器 使用）

### full\_audit:priority

syslog 严重性（优先）级别

* emerg: Emergency（紧急）- 0
* alert: Alerts （报警）- 1
* crit: Critical (关键）- 2
* err: Errors （错误）- 3
* warn: Warnings （警告）- 4
* notice: Notification （通知）- 5
* info: Information （消息）- 6
* debug: Debugging （调试）- 7



## 操作列表



* `chdir` :
* `chflags` :
* `chmod` :
* `chown` :
* `close` :
* `closedir` :
* `connect` :
* `copy_chunk_send` :
* `copy_chunk_recv` :
* `disconnect` :
* `disk_free` :
* `fchmod` :
* `fchown` :
* `fget_nt_acl` :
* `fgetxattr` :
* `flistxattr` :
* `fremovexattr` :
* `fset_nt_acl` :
* `fsetxattr` :
* `fstat` :
* `fsync` :
* `ftruncate` :
* `get_compression` :
* `get_nt_acl` :
* `get_quota` :
* `get_shadow_copy_data` :
* `getlock` :
* `getwd` :
* `getxattr` :
* `kernel_flock` :
* `link` :
* `linux_setlease` :
* `listxattr` :
* `lock` :
* `lseek` :
* `lstat` :
* `mkdir` :
* `mknod` :
* `open` :
* `opendir` :
* `pread` :
* `pwrite` :
* `read` :
* `readdir` :
* `readlink` :
* `realpath` :
* `removexattr` :
* `rename` :
* `rewinddir` :
* `rmdir` :
* `seekdir` :
* `sendfile` :
* `set_compression` :
* `set_nt_acl` :
* `set_quota` :
* `setxattr` :
* `snap_check_path` :
* `snap_create` :
* `snap_delete` :
* `stat` :
* `statvfs` :
* `symlink` :
* `sys_acl_delete_def_file` :
* `sys_acl_get_fd` :
* `sys_acl_get_file` :
* `sys_acl_set_fd` :
* `sys_acl_set_file` :
* `telldir` :
* `unlink` :
* `utime` :
* `write` :

