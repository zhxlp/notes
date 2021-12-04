# FPM 打包 nginx rpm 包

## 安装 FPM

- 安装依赖

  ```bash
  yum install -y  ruby rubygems ruby-devel gcc rpm-build
  ```

- 配置 RubyGems 源

  ```bash
  # 移除默认源
  gem sources --remove https://rubygems.org/
  # 配置阿里源
  gem sources -a https://mirrors.aliyun.com/rubygems/
  ```

- 安装 fpm

  ```bash
  gem install fpm
  ```

## 编译 nginx

- 下载 nginx 源码包

  ```bash
  wget http://nginx.org/download/nginx-1.18.0.tar.gz
  ```

- 安装依赖

  ```bash
  yum install -y pcre-devel openssl-devel
  ```

- 解压

  ```bash
  tar xvf nginx-1.18.0.tar.gz
  cd nginx-1.18.0
  ```

- 编译

  ```bash
  ./configure --with-http_stub_status_module  --with-http_ssl_module
  make
  ```

## 打包

- 安装到临时目录

  ```bash
  rm -rf /tmp/nginx
  mkdir /tmp/nginx
  make DESTDIR=/tmp/nginx install
  ```

- 打包

  ```bash
  cd ..
  fpm -s dir -t rpm -n nginx -v 1.18.0 -C /tmp/nginx -f -d 'pcre,openssl'
  ```

## 其它命令

- 查看 rpm 执行的脚本

  ```bash
  rpm -qp --scripts nginx-1.18.0-1.x86_64.rpm
  ```

- 查看 rpm 包的依赖

  ```bash
  rpm -qpR nginx-1.18.0-1.x86_64.rpm
  ```

- 查看 rpm 包中的内容

  ```bash
  rpm -qpl nginx-1.18.0-1.x86_64.rpm
  ```

- 查看 rpm 包信息

  ```bash
  rpm -qpi nginx-1.18.0-1.x86_64.rpm
  ```

- fpm 帮助

  ```properties
  # fpm -h #查看命令的帮助，下面对常用的参数进行简单的说明
  # -s：指定源类型
  # -t：指定目标类型
  # -n：指定名字
  # -v：指定版本号
  # -C：指定打包的相对路径
  # -d：指定依赖于哪些包
  # -f：第二次打包时目录下如果有同名安装包存在，则覆盖它
  # -p：输出的安装包的目录，不想放在当前目录下就需要指定
  # --iteration 修订版本
  # --before-install  软件包安装之前所要运行的脚本
  # --after-install  软件包安装之后所要运行的脚本
  # --before-remove  软件包卸载之前所要运行的脚本
  # --after-remove   软件包卸载之后所要运行的脚本
  # --vendor  供应商 例 zhxlp
  # -m：维护者 例 'zhxlp <zhxlp@zhxlp.com>'
  # --url  软件包相关信息链接
  # --license  协议
  # --description 描述
  # --rpm-sumarry 简介
  # --rpm-dist 定义系统的迭代版本，el6表示centos6，el7表示centos7
  ```
