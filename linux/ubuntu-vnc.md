# Ubuntu VNC

*   安装 Xfce 桌面环境

    ```bash
    sudo apt install xfce4 xfce4-goodies
    ```
*   安装 TightVNC 服务器

    ```bash
    sudo apt install tightvncserver
    ```
*   设置密码

    ```bash
    vncpasswd
    # root@local:~# vncserver
    # You will require a password to access your desktops.
    # Password:
    # Verify:
    # Would you like to enter a view-only password (y/n)? n
    ```
*   配置 vnc 服务器

    ```bash
    # 修改配置文件
    echo '#!/bin/sh
    export SHELL="/bin/bash"
    unset SESSION_MANAGER
    unset DBUS_SESSION_BUS_ADDRESS
    startxfce4 &

    [ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
    [ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
    xsetroot -solid grey
    ' > ~/.vnc/xstartup
    sudo chmod +x ~/.vnc/xstartup
    ```
*   注册服务

    ```bash
    echo '[Unit]
    Description=Start TightVNC server at startup
    After=syslog.target network.target

    [Service]
    Type=forking
    User=root
    Group=root
    WorkingDirectory=/root/

    PIDFile=/root/.vnc/%H:%i.pid
    ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
    ExecStart=/usr/bin/vncserver :%i
    ExecStop=/usr/bin/vncserver -kill :%i

    [Install]
    WantedBy=multi-user.target
    ' > /etc/systemd/system/vncserver@.service

    sudo systemctl daemon-reload
    sudo systemctl enable vncserver@1.service
    sudo systemctl restart vncserver@1.service
    ```
