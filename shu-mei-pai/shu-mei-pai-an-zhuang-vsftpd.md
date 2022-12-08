# 树莓派安装 vsftpd

*   安装依赖包

    ```bash
    apt install vsftpd libpam-pwdfile apache2-utils
    ```
*   编辑配置文件`/etc/vsftpd.conf`

    ```properties
    # 独立模式运行，由vsftpd自己监听和处理连接请求
    listen=YES
    # 禁止匿名用户登录
    anonymous_enable=NO
    # 允许本地用户(包含虚拟用户)登录
    local_enable=YES
    # 本地用户写权限
    write_enable=YES
    # 本地用户掩码
    local_umask=022
    # 匿名用户掩码
    anon_umask=022
    # 本地用户(包含虚拟用户)主目录
    local_root=/var/www
    # 是否将所有用户限制在主目录,YES为启用 NO禁用.
    chroot_local_user=YES
    # 是否允许在主目录有写权限,YES:有写权限,NO:没有写权限(并且要去除主目录写权限 chomd a-w)
    allow_writeable_chroot=YES
    # 隐藏文件用户和组信息,显示为ftp
    hide_ids=YES
    # 运行vsftpd需要的非特权系统用户,程序运行用户
    nopriv_user=vsftpd

    # 使用utf-8字符集
    utf8_filesystem=YES

    # vsftpd将使用的PAM服务的名称 /etc/pam.d/vsftpd
    pam_service_name=vsftpd
    #启用虚拟账户
    guest_enable=YES
    # 虚拟用户权限,YES:与本地用户权限相同.NO:与匿名用户权限相同
    virtual_use_local_privs=NO
    # 虚拟用户映射的真实用户
    guest_username=vsftpd
    # 用户配置目录,可以为每个用户配置不同配置
    user_config_dir=/etc/vsftpd/user_conf

    # 日志
    xferlog_enable=YES
    xferlog_std_format=YES
    xferlog_file=/var/log/xferlog
    dual_log_enable=YES
    vsftpd_log_file=/var/log/vsftpd.log
    ```
*   创建系统用户`vsftpd`

    ```bash
    useradd --home /home/vsftpd --gid nogroup -m --shell /bin/false vsftpd
    ```
*   创建虚拟用户

    ```bash
    mkdir /etc/vsftpd
    htpasswd -c -p -b /etc/vsftpd/ftpd.passwd admin $(openssl passwd -1 -noverify)
    ```

````
  > **htpasswd**
  >
  > * -c  创建一个加密文件
  >
  > * -p   不对密码进行进行加密，即明文密码
  >
  > * -b   在命令行中一并输入用户名和密码而不是根据提示输入密码
  >
  > **openssl passwd**
  >
  > * -1    md5-crypt算法加密(结果格式: $1$salt$encrypted )
  > * -noverify    不进行验证

* 编辑`/etc/pam.d/vsftpd`文件

  ```properties
  auth required pam_pwdfile.so pwdfile /etc/vsftpd/ftpd.passwd
  account required pam_permit.so
````

*   创建`admin`用户配置文件`/etc/vsftpd/user_conf/admin`

    ```properties
    # 本地用户(包含虚拟用户)主目录
    local_root=/var/www
    # 匿名文件掩码
    anon_umask=022
    # 只允许匿名用户下载自己可读的文件
    anon_world_readable_only=YES
    # 允许匿名用户在特定条件下创建新目录
    anon_mkdir_write_enable=YES
    # 允许匿名用户执行除上载和创建目录之外的写入操作,例如删除和重命名
    anon_other_write_enable=YES
    # 允许匿名用户在特定条件下上载文件
    anon_upload_enable=YES
    ```
*   参考链接

    https://askubuntu.com/questions/575523/how-to-setup-virtual-users-for-vsftpd-with-access-to-a-specific-sub-directory https://www.cnblogs.com/miclesvic/articles/10437213.html https://wiki.ubuntu.org.cn/Vsftpd虚拟用户设置 https://www.cnblogs.com/wangliangblog/p/7325819.html
