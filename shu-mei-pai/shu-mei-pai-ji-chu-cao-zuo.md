# 树莓派基础操作

## 更换 apt 源

- 编辑`/etc/apt/sources.list`文件，将地址变更为`http://mirrors.ustc.edu.cn/raspbian/raspbian/`

  ```properties
  deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
  deb-src http://mirrors.ustc.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
  ```

- 编辑`/etc/apt/sources.list.d/raspi.list`,将地址变更为`http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/`

  ```properties
  deb http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/ buster main
  deb-src http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/ buster main
  ```

## 更换 pip 源

- 更换阿里 pip 源

  ```bash
  mkdir ~/.pip
  cat > ~/.pip/pip.conf << EOF
  [global]
  index-url = https://mirrors.aliyun.com/pypi/simple/
  [install]
  trusted-host=mirrors.aliyun.com
  EOF
  ```

## 配置 DNS 缓存

- 安装软件包

  ```bash
  apt install nscd
  ```

- 修改配置文件`/etc/nscd.conf`，关闭不需要的缓存

  nscd 默认缓存 passwd, group, hosts, services, or netgroup 五个
