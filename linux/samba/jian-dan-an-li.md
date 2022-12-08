# 配置

## 需求

#### 用户

| 用户   | 账号         | 用户组        |
| ---- | ---------- | ---------- |
| 张三   | zhangsan   | zhangsan   |
| 李四   | lisi       | lisi       |
| 公共数据 | publicdata | publicdata |

#### 目录

| 名称   | 路径         | 权限   | 用户:组                  |
| ---- | ---------- | ---- | --------------------- |
| 公共数据 | /data/公共数据 | 0770 | publicdata:publicdata |
| 张三   | /data/张三   | 0770 | zhangsan:zhangsan     |
| 李四   | /data/李四   | 0770 | lisi:lisi             |

#### 用户与目录

| 用户 | 目录      |
| -- | ------- |
| 张三 | 张三、公共数据 |
| 李四 | 李四、公共数据 |
|    |         |



## 步骤

### 创建linux用户

```bash
# 创建用户
# -M 不创建用户目录
# -s /sbin/nologin 不登录
# -c 备注
useradd -M -s /sbin/nologin -c '张三' zhangsan
useradd -M -s /sbin/nologin -c '李四' lisi
useradd -M -s /sbin/nologin -c '公共数据' publicdata

# 将用户添加到 公共数据组
usermod -a -G publicdata zhangsan
usermod -a -G publicdata lisi
```

### 创建samba用户

```bash
# 创建samba用户
# echo -e 'password\npassword' 打印两遍密码并通过管道传输到 pdbedit
# smbpasswd -a 用户名  添加用户 
echo -e 'password\npassword' | smbpasswd -a zhangsan
echo -e 'password\npassword' | smbpasswd -a lisi
echo -e 'password\npassword' | smbpasswd -a publicdata
```

### 创建目录

```bash
# 创建目录
mkdir -p /data/公共数据
mkdir -p /data/张三
mkdir -p /data/李四

# 设置权限
chmod 0770 /data/公共数据
chmod 0770 /data/张三
chmod 0770 /data/李四

# 更改用户及用户组
chown publicdata:publicdata /data/公共数据
chown zhangsan:zhangsan /data/张三
chown lisi:lisi /data/李四
```

### samba配置一

修改 `/etc/samba/smb.conf`

```properties
[global]
        workgroup = SAMBA
        # 用户认证
        security = user
        passdb backend = tdbsam
        printing = cups
        printcap name = cups
        load printers = yes
        cups options = raw

[公共数据]
        # 描述
        comment = 公共数据
        # 路径
        path = /data/公共数据
        # 是否可用
        available = yes
        # 在上级目录中显示
        browseable = yes
        # 创建文件权限
        create mode = 0660
        # 创建目录权限
        directory mode = 0770
        # 所有者用户组
        force group = publicdata
        # 所有者
        force user = publicdata
        # 登陆校验(用户、用户组)列表
        valid users = @publicdata
        # 读取权限(用户、用户组)列表
        read list = @publicdata
        # 写入权限(用户、用户组)列表
        write list = @publicdata
        
[张三]
        # 描述
        comment = 张三个人数据
        # 路径
        path = /data/张三
        # 是否可用
        available = yes
        # 在上级目录中显示
        browseable = yes
        # 创建文件权限
        create mode = 0660
        # 创建目录权限
        directory mode = 0770
        # 所有者用户组
        force group = zhangsan
        # 所有者
        force user = zhangsan
        # 登陆校验(用户、用户组)列表
        valid users = @zhangsan
        # 读取权限(用户、用户组)列表
        read list = @zhangsan
        # 写入权限(用户、用户组)列表
        write list = @zhangsan
[李四]
        # 描述
        comment = 李四个人数据
        # 路径
        path = /data/李四
        # 是否可用
        available = yes
        # 在上级目录中显示
        browseable = yes
        # 创建文件权限
        create mode = 0660
        # 创建目录权限
        directory mode = 0770
        # 所有者用户组
        force group = lisi
        # 所有者
        force user = lisi
        # 登陆校验(用户、用户组)列表
        valid users = @lisi
        # 读取权限(用户、用户组)列表
        read list = @lisi
        # 写入权限(用户、用户组)列表
        write list = @lisi

```

#### 重载配置

```bash
systemctl restart smb.service
```

#### 结果

<figure><img src="../../.gitbook/assets/Snipaste_2022-11-27_17-14-38.png" alt=""><figcaption></figcaption></figure>

### samba配置二

#### smb.conf

修改 `/etc/samba/smb.conf`

```properties
[global]
        workgroup = SAMBA
        # 用户认证
        security = user
        passdb backend = tdbsam
        printing = cups
        printcap name = cups
        load printers = yes
        cups options = raw
        # 更具用户导入不同配置, %U 用户名
        config file = /etc/samba/conf.d/user.conf.%U

```

#### 创建子配置目录

```bash
mkdir -p /etc/samba/conf.d
```

#### 公共数据目录配置

修改 `/etc/samba/conf.d/directory.conf.公共数据`

```properties
[公共数据]
        # 描述
        comment = 公共数据
        # 路径
        path = /data/公共数据
        # 是否可用
        available = yes
        # 在上级目录中显示
        browseable = yes
        # 创建文件权限
        create mode = 0660
        # 创建目录权限
        directory mode = 0770
        # 所有者用户组
        force group = publicdata
        # 所有者
        force user = publicdata
        # 登陆校验(用户、用户组)列表
        valid users = @publicdata
        # 读取权限(用户、用户组)列表
        read list = @publicdata
        # 写入权限(用户、用户组)列表
        write list = @publicdata
```

#### 张三目录配置

修改 `/etc/samba/conf.d/directory.conf.张三`

```properties
[张三]
        # 描述
        comment = 张三个人数据
        # 路径
        path = /data/张三
        # 是否可用
        available = yes
        # 在上级目录中显示
        browseable = yes
        # 创建文件权限
        create mode = 0660
        # 创建目录权限
        directory mode = 0770
        # 所有者用户组
        force group = zhangsan
        # 所有者
        force user = zhangsan
        # 登陆校验(用户、用户组)列表
        valid users = @zhangsan
        # 读取权限(用户、用户组)列表
        read list = @zhangsan
        # 写入权限(用户、用户组)列表
        write list = @zhangsan
```

#### 李四目录配置

修改 `/etc/samba/conf.d/directory.conf.李四`

```properties
[李四]
        # 描述
        comment = 李四个人数据
        # 路径
        path = /data/李四
        # 是否可用
        available = yes
        # 在上级目录中显示
        browseable = yes
        # 创建文件权限
        create mode = 0660
        # 创建目录权限
        directory mode = 0770
        # 所有者用户组
        force group = lisi
        # 所有者
        force user = lisi
        # 登陆校验(用户、用户组)列表
        valid users = @lisi
        # 读取权限(用户、用户组)列表
        read list = @lisi
        # 写入权限(用户、用户组)列表
        write list = @lisi
```

#### 张三用户配置

修改 `/etc/samba/conf.d/user.conf.zhangsan`

```properties
[global]
    include = /etc/samba/conf.d/directory.conf.公共数据
    include = /etc/samba/conf.d/directory.conf.张三
```

#### 李四用户配置

修改 `/etc/samba/conf.d/user.conf.lisi`

```properties
[global]
    include = /etc/samba/conf.d/directory.conf.公共数据
    include = /etc/samba/conf.d/directory.conf.李四
```

#### 重载配置

```bash
systemctl restart smb.service
```

#### 结果

<figure><img src="../../.gitbook/assets/Snipaste_2022-11-27_18-20-13.png" alt=""><figcaption></figcaption></figure>

#### 重写导入配置

示例：张三公共数据只读权限

修改 `/etc/samba/conf.d/user.conf.zhangsan`

```properties
[global]
    include = /etc/samba/conf.d/directory.conf.公共数据
    include = /etc/samba/conf.d/directory.conf.张三

[公共数据]
    # 写入权限(用户、用户组)列表
    write list = 
```
