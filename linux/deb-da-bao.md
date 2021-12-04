# deb 打包

## 参考链接

https://www.ubuntukylin.com/ukd/home/deb2.php

https://www.debian.org/doc/manuals/maint-guide/start.zh-cn.html

https://blog.serverdensity.com/how-to-create-a-debian-deb-package/

https://www.downeyboy.com/2019/05/28/mk\_deb\_pack\_series\_0/

https://blog.csdn.net/steveyg/article/details/78059276 https://www.cnblogs.com/whwywzhj/p/11214771.html

## debian 目录内容

* control： control 决定着 deb 包的包名、编译依赖和运行依赖
* changelog： 供版本修改信息
* copyright： 版权和许可
* preinst：Debian 软件包(".deb")解压前执行的脚本，为正在被升级的包停止相关服务，直到升级或安装完成(成功后执行 'postinst' 脚本)
* postinst：主要完成软件包(".deb")安装完成后所需的配置工作。通常，postinst 脚本要求用户输入， 和/或警告用户如果接受默认值，应该记得按要求返回重新配置这个软件。一个软件包安装或升级完成后，postinst 脚本驱动命令，启动或重起相应的服务。
* prerm：停止一个软件包的相关进程，要卸载软件包的相关文件前执行
* postrm：修改相关文件或连接，和/或卸载软件包所创建的文件
* conffiles：配置文件列表，卸载时文件不会被删除，升级时会提示是否覆盖
*   md5sums：该文件是包中包含的所有文件的 MD5 校验和的列表，这些文件实际上将被提取到系统中。使用命令组合可以很容易地生成

    ```bash
    find . -type f ! -regex '.*.hg.*' ! -regex '.*?debian-binary.*' ! -regex '.*?DEBIAN.*' -printf '%P ' | xargs md5sum > DEBIAN/md5sums
    ```

| deb script         |                      |               |
| ------------------ | -------------------- | ------------- |
| install            | upgrade              | uninstall     |
| preinst install    | 旧包prerm upgrade      | prerm remove  |
| 安装文件               | 新包preinst upgrade    | 删除文件          |
| postinst configure | 安装新文件                | postrm remove |
|                    | 旧包postrm upgrade     |               |
|                    | 新包postinst configure |               |

[实例](test-deb.tar.gz)

## control

这个文件包含了很多供 **dpkg**、**dselect**、**apt-get**、**apt-cache**、**aptitude** 等包管理工具进行管理时所使用的许多变量。这些变量均在 [Debian Policy Manual, 5 "Control files and their fields"](http://www.debian.org/doc/debian-policy/ch-controlfields.html) 中被定义。

二进制程序包

* [Package](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-package) (必填)： 软件包名称 ，必须仅由小写字母（a-z），数字（0-9），加号（+）和减号（-）和句点（.）组成。它们必须至少两个字符长，并且必须以字母数字字符开头。
* [Version](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-version) (必填)：软件包的版本号。格式为： \[epoch:]upstream\_version\[-debian\_revision]。
* [Architecture](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-architecture) (必填)：软件包结构，如 i386, amd64,m68k, sparc, alpha, powerpc,any 等
* [Maintainer](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-maintainer) (必填)：软件包维护者的姓名和电子邮件地址。名称必须首先出现，然后是尖括号内的电子邮件地址<>（采用 RFC822 格式）
* [Description](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-description) (必填)：对二进制包的描述，该描述由两部分组成，即概要或简短描述以及详细描述
* [Section](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-section) (推荐)： 申明软件的类别，常见的有 utils, net, mail, text, devel 等
* [Priority](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-priority) (推荐)：申明软件对于系统的重要程度，如 required, standard, optional, extra 等
* [Source](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-source)： 源代码包的名称
* [Essential](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-essential)：申明是否是系统最基本的软件包（选项为 yes/no），如果是的话，这就表明该软件是维持系统稳定和正常运行的软件包，不允许任何形式的卸载（除非进行强制性的卸载）
* [Depends](https://www.debian.org/doc/debian-policy/ch-relationships.html#s-binarydeps)：声明了绝对依赖性
* [Installed-Size](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-installed-size)： 安装指定软件包所需的磁盘空间总量的估计值 ,单位字节
*   [Homepage](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-homepage)：软件包的网站的 URL

    ```properties
    Package:
    Version:
    Architecture:
    Maintainer:
    Description:

    Section:
    Priority:
    Source:
    Essential:
    Depends:
    Installed-Size:
    Homepage:
    ```

    ### changelog

    每个源软件包都必须包含 Debian 变更日志文件 `debian/changelog`。该文件的 Debian 版本中的更改应简要说明。

    * package: 软件包名称
    * version:版本号
    * distribution(s) ： 上载此版本时应在其中安装的发行版 ，一般用 unstable
    * urgency: 从以前的版本升级到此版本的重要性。它由值的单个关键字取一个的`low`，`medium`，`high`，`emergency`，或`critical`

    ```properties
    package (version) distribution(s); urgency=urgency
      * 更新内容1
      * 更新内容2
     -- root <root@unknown>  Mon, 23 Dec 2019 10:02:38 +0800
    ```

    注：date 可以使用命令 date -R 获取
