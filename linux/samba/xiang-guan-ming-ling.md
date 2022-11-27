# 相关命令

## smbpasswd

samba密码管理工具

### 修改密码

```bash
smbpasswd -s 用户名
```

### 添加用户

```bash
smbpasswd -a 用户名
```

### 禁用用户

```
smbpasswd -d 用户名
```

### 启用用户

```
smbpasswd -e 用户名
```

### 空密码

```
smbpasswd -n 用户名
```

## smbstatus

samba 连接状态



## pdbedit

samba用户数据库管理工具

### 用户列表

```bash
pdbedit -L

# 详细信息
pdbedit -Lv
```

### 添加用户

```bash
pdbedit -a -u 用户名
```

### 删除用户

```
pdbedit -x -u 用户名
```

### 设置账户控制属性

```
# N: 无需密码
# D: 账户已禁用
# H: 要求主目录
# T: 其他账户的临时副本
# U: 普通用户账户
# M: MNS 登录用户帐户
# W: 工作站信任帐户
# S: 服务器信任帐户
# L: 自动锁定
# X: 密码不过期
# I: 域信任帐户

# 设置密码不过期
pdbedit -u 用户名 -c "[X]"

# 禁用用户
pdbedit -u 用户名 -c "[XD]"
```

