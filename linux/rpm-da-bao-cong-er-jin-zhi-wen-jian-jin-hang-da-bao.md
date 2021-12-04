# rpm 打包-从二进制文件进行打包

## 简介

## 文件

- webdav

  访问 https://github.com/hacdias/webdav/releases 下载 linux-amd64-webdav.tar.gz

  获取压缩包里面的 webdav 文件

- webdav.service

  ```properties
  [Unit]
  Description=WebDAV server
  After=network.target

  [Service]
  Type=simple
  User=root
  ExecStart=/usr/bin/webdav --config /etc/webdav.config
  Restart=on-failure

  [Install]
  WantedBy=multi-user.target
  ```

- config.yaml

  ```properties
  # Server related settings
  address: 0.0.0.0
  port: 8888
  auth: true
  tls: false
  cert: cert.pem
  key: key.pem
  prefix: /

  users:
    - username: admin
      password: admin
      modify: true
      scope: /data/webdav
  ```

- webdav.spec

  ```properties
  Name:           webdav
  Summary:        WebDAV for Linux
  Version:        3.0.0
  Release:        linux
  BuildArch:      x86_64
  Group:          Applications/Internet
  URL:            https://github.com/hacdias/webdav
  Distribution:   Linux/x64
  Packager:       Zhxlp<zhxlp@zhxlp.com>
  License:        MIT

  %description
  A WebDAV server program

  %install
  cp -r %{_builddir}/* %{buildroot}/

  # 安装前
  %pre
  systemctl stop webdav.service || true

  # 安装后
  %post
  mkdir -p /data/webdav || true
  chmod 777 /data/webdav || true
  systemctl daemon-reload || true

  # 卸载前
  %preun
  systemctl stop webdav.service || true

  # 卸载后
  %postun
  systemctl daemon-reload || true

  %clean
  rm -rf %{buildroot}

  # 添加到安装包的文件
  %files
  %attr(755,root,root) /usr/bin/webdav
  /usr/bin/webdav
  /etc/webdav.config
  /usr/lib/systemd/system/webdav.service
  ```

- 脚本执行顺序及参数

  <table>
      <tr>
          <td colspan="3">rpm script</td>
      </tr>
      <tr>
          <td>install</td>
          <td>upgrade</td>
          <td>uninstall</td>
      </tr>
      <tr>
          <td>%pre 1</td>
          <td>运行新包的%pre 2</td>
          <td>%preun 0</td>
      </tr>
      <tr>
          <td>安装文件</td>
          <td>安装新文件</td>
          <td>删除文件</td>
      </tr>
      <tr>
          <td>%post 1</td>
          <td>运行新包的%post 2</td>
          <td>%postun 0</td>
      </tr>
      <tr>
          <td></td>
          <td>运行旧包的%preun 1</td>
          <td></td>
      </tr>
      <tr>
          <td></td>
          <td>删除新文件未覆盖的任何旧文件</td>
          <td></td>
      </tr>
      <tr>
          <td></td>
          <td>运行旧包的%postun 1</td>
          <td></td>
      </tr>
  </table>

- build.sh

  ```properties
  #!/bin/bash
  set -e
  version=3.0.0
  if [ -L $0 ]; then
      BASE_DIR=`dirname $(readlink $0)`
  else
      BASE_DIR=`dirname $0`
  fi
  bashpath=$(cd $BASE_DIR; pwd)
  cd $bashpath

  sed -i "s#Version: .*#Version: $version#" $bashpath/SPECS/webdav.spec
  echo '%_topdir' "$bashpath" > ~/.rpmmacros
  rpmbuild -bb $bashpath/SPECS/webdav.spec
  ```

## 步骤

- 安装依赖

  ```bash
  yum install -y rpm-build
  ```

- 创建目录结构

  ```bash
  mkdir {BUILD,BUILDROOT,SPECS,RPMS}
  ```

- 复制文件到制定目录

  ```properties
  ./
  ├── BUILD
  │   ├── etc
  │   │   └── webdav.config
  │   └── usr
  │       ├── bin
  │       │   └── webdav
  │       └── lib
  │           └── systemd
  │               └── system
  │                   └── webdav.service
  ├── BUILDROOT
  ├── build.sh
  ├── RPMS
  └── SPECS
      └── webdav.spec
  ```

- 打包

  ```bash
  bash build.sh
  ```

  打包好的 rpm 包生成在 RPMS 目录

## 参考

https://rpm-packaging-guide.github.io/

https://fedoraproject.org/wiki/How_to_create_a_GNU_Hello_RPM_package/zh-cn

https://blog.csdn.net/get_set/article/details/53453320

https://docs.fedoraproject.org/en-US/packaging-guidelines/Scriptlets/
https://www.golinuxhub.com/2018/05/how-to-execute-script-at-pre-post-preun-postun-spec-file-rpm/
