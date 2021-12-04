# Ubuntu18.04-RealVNC 安装

- 使用 LightDM 管理

  使用 LightDM 管理，使 VNC 支持桌面不登录也可以打开。

  ```bash
  apt install lightdm
  ```

- 设置 root 用户登录桌面

  编辑`/usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf`文件，添加`greeter-show-manual-login=true`

- 安装 RealVNC

  访问官网下载软件包

  ```bash
  # 安装软件包
  dpkg -i VNC-Server-6.5.0-Linux-x64.deb
  # 设置密钥
  sudo vnclicense -add WHJRK-UXY7V-Q34M9-CZU8L-8KGFA
  ```

- 启动服务

  ```bash
  systemctl start vncserver-x11-serviced.service
  ```
