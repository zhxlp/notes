# Centos7重置密码

在开机引导界面输入`e`编辑开机引导项，在Linux16一行末尾添加 rd.break，按快捷键Ctrl+X,用户修改后的引导项启动电脑.

![引导界面](CentOS7重置密码/image/1540262747784.png)

![修改引导](CentOS7重置密码/image/1540262941553.png)

开机后输入如下命令进行密码修改

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd
touch /.autorelabel
exit
rebas
```

![修改后](CentOS7重置密码/image/1540263618643.png)
