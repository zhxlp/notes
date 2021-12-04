# CentOS7.3 编译安装 Python3.6

- 安装依赖

  ```bash
  yum install -y gcc-c++ valgrind-devel systemtap-sdt-devel \
  bzip2-devel ncurses-devel gdbm-devel sqlite-devel \
  openssl-devel readline-devel zlib-devel xz-devel tk-devel
  ```

- 下载源码包,解压并进入解压目录

  ```bash
  wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz
  tar xvf Python-3.6.8.tgz
  cd Python-3.6.8
  ```

- 编译并安装

  ```bash
  ./configure  --prefix=/usr/local/ \
  --enable-ipv6 \
  --enable-shared \
  --with-computed-gotos=yes \
  --with-dbmliborder=gdbm:ndbm:bdb \
  --with-system-expat \
  --with-system-ffi \
  --enable-loadable-sqlite-extensions \
  --with-dtrace \
  --with-valgrind \
  --with-ensurepip \
  --enable-optimizations

  make && make install
  ```

  > --enable-shared 安装共享库,共享库在使用用其他需调用用 python 的软件时会用用到
  >
  > --enable-optimizations 启动性能优化，会增加编译时间，提高运行速度

- 配置动态库

  ```bash
  echo '/usr/local/lib' > /etc/ld.so.conf.d/python3.6.conf
  ldconfig
  ```

- 修改 pip 源

  ```bash
  mkdir ~/.pip
  cat > ~/.pip/pip.conf << EOF
  [global]
  index-url = https://mirrors.aliyun.com/pypi/simple/
  [install]
  trusted-host=mirrors.aliyun.com
  EOF
  ```

- 安装虚拟环境，并配置

  ```bash
  pip3.6 install virtualenvwrapper
  cat >> /etc/profile << EOF
  export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3.6
  export WORKON_HOME=/opt/.virtualenv
  source /usr/local/bin/virtualenvwrapper.sh
  EOF
  source /etc/profile
  ```

- 虚拟环境命令

  ```bash
  # mkvirtualenv 创建虚拟环境
  # 创建名为 dev1的虚拟环境
  # mkvirtualenv dev1
  # 创建名为 dev2的虚拟环境，指定python版本2.7
  # mkvirtualenv -p python2.7 dev2

  # lsvirtualenv 查看虚拟环境列表

  # rmvirtualenv 删除虚拟环境

  # workon 进入虚拟环境

  # deactivate 退出虚拟环境
  ```
