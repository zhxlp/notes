# Openstack-Windows 镜像制作

* 依赖文件
  * [virtio-win.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/) windows 驱动
  * [qemu-ga](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-qemu-ga/) qemu 代理程序,用于修改虚拟机密码
  * [CloudbaseInit](https://cloudbase.it/downloads/CloudbaseInitSetup\_Stable\_x64.msi) 虚拟机初始化操作，如修改主机名等
*   创建虚拟机

    ```bash
    virt-install --name Windows7_Ultimate_x64 \
    --vcpus 1 \
    --memory 2048 \
    --disk path=/var/lib/libvirt/images/Windows7_Ultimate_x64.qcow2,size=40,format=qcow2,bus=scsi \
    --disk path=/data/iso/cn_windows_7_ultimate_with_sp1_x64_dvd_u_677408.iso,device=cdrom,readonly=on \
    --disk path=/data/iso/virtio-win-0.1.141.iso,device=cdrom,readonly=on \
    --channel unix,target_type=virtio,name=org.qemu.guest_agent.0 \
    --controller scsi,model=virtio-scsi \
    --network bridge=br0,model=virtio \
    --graphics vnc,listen=0.0.0.0 \
    --noautoconsole

    # 查看vnc端口
     virsh vncdisplay Windows7_Ultimate_x64
    ```
*   加载硬盘驱动

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_11-47-11.png)

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_11-47-32.png)

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_11-52-42.png)

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_11-53-09.png)
* 正常安装操作系统
* 跳过用户创建，直接启动 Administrator 用户
  *   使用快捷键`Shift + F10`打开 cmd，输入`compmgmt.msc`打开计算机管理。

      !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_12-11-08.png)
  *   在计算机管理界面启用`Administrator`用户，并设置用户密码

      !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_12-12-43.png)
  *   再次打开`cmd`，输入`taskmgr`,结束`msobe.exe`程序

      !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_12-17-05.png)
*   安装未识别设备驱动

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_12-22-10.png)

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_12-22-31.png)

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_12-53-36.png)

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_12-54-02.png)
* 重启虚拟机
* 开启远程桌面，防火墙放行远程桌面
*   关闭密码安全策略

    打开策略组编辑器`gpedit.msc`,进入 【计算机配置】-【Windows 设置】-【安全设置】-【账号策略】-【密码策略】,禁用“密码必须符合复杂性要求”

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_14-08-26.png)
*   安装 qemu-ga 程序

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_13-07-16.png)
*   安装 Cloudinit

    !\[]\(../.gitbook/assets/Snipaste\_2019-07-16\_13-10-18.png)
* 删除软件包，关闭虚拟机
*   转换镜像格式为`raw`

    ```bash
    qemu-img convert -O raw Windows7_Ultimate_x64.qcow2 Windows7_Ultimate_x64.raw
    ```
