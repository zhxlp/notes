# Ubuntu 20.04 ARM64 修改PAGE\_SIZE 64k

## 环境

```properties
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.5 LTS
Release:        20.04
Codename:       focal

$ uname -a
Linux ikmak-build 5.4.0-132-generic #148-Ubuntu SMP Mon Oct 17 16:03:31 UTC 2022 aarch64 aarch64 aarch64 GNU/Linux
```



## 步骤

### 启用src源

编辑 `/etc/apt/sources.list`  ,取消如下行的注释

```properties
deb-src http://xxxxxx disco main
deb-src http://xxxxxx disco-updates main
```

更新源

```bash
sudo apt update
```

### 安装依赖

```
sudo apt-get build-dep -y linux linux-image-$(uname -r)
sudo apt-get install -y libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm fakeroot
```

### 创建编译目录

```bash
mkdir linux
cd linux
```

### 下载源码

```
apt-get source linux-image-unsigned-$(uname -r)
```

### 修改配置

```bash
cd linux-5.4.0
chmod a+x debian/rules
chmod a+x debian/scripts/*
chmod a+x debian/scripts/misc/*
# 清理
LANG=C fakeroot debian/rules clean
# 开始配置了，注意提示
LANG=C make menuconfig
# 修改 page size 64kb
# 选择 Kernel Features -> Page size -> 选择 64k -> ESC ESC ESC ESC YES
```

### 编译

```bash
LANG=C make
```

### 安装

```bash
sudo make modules_install
sudo make install
```

### 重启系统



## 参考

{% embed url="https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel" %}

{% embed url="https://www.ab62.cn/article/1191.html" %}



















