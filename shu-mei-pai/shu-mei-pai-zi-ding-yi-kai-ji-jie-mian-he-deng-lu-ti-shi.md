# 树莓派自定义开机界面和登录提示

## 开机界面

- 系统信息

  ```properties
  root@raspberrypi:~# lsb_release -a
  No LSB modules are available.
  Distributor ID:	Raspbian
  Description:	Raspbian GNU/Linux 10 (buster)
  Release:	10
  Codename:	buster
  ```

- 禁止颜色测试

  编辑`/boot/config.txt` 中添加 `disable_splash=1`

- 禁用角落 LOGO

  编辑`/boot/cmdline.txt`添加`logo.nologo`

- 禁用开机显示日志

  编辑`/boot/cmdline.txt`添加`quiet`

- 安装图片查看工具`fbi`

  ```bash
  apt install fbi
  ```

- 定义开机启动界面服务

  创建`/etc/systemd/system/splashscreen.service`

  ```properties
  [Unit]
  Description=Splashscreen
  DefaultDependencies=no
  After=basic.target
  [Service]
  ExecStart=/usr/bin/fbi -d /dev/fb0 --noverbose -a /boot/splash.jpg
  StandardInput=tty
  StandardOutput=tty
  Restart=always
  StartLimitInterval=0
  [Install]
  WantedBy=sysinit.target
  ```

- 上传图片

  上传开机界面图到`/boot/`目录，并命名为`splash.jpg`

- 设置服务开机自启并禁至用户通过 tty1 登录

  ```bash
  systemctl enable splashscreen.service
  systemctl disable getty@tty1.service
  ```

- 设置默认分辨率

  当没有设置默认分变率时，在没有连接显示器的情况下开机，开机之后连接显示器不会显示图像界面，所有设置一下默认分辨率。

  输入命令`raspi-config`，选择`Advanced Options`，再选择`Resolution`,在页面中选择默认分辨率

## 登录提示

- /etc/issue 本地端登录前显示信息文件
- /etc/issue.net 网络端登录前显示信息文件
- /etc/motd 登陆后显示信息文件
