# FPM打包nginx rpm包

## 安装FPM <a href="#an-zhuang-fpm" id="an-zhuang-fpm"></a>

* 安装依赖

```bash
yum install -y  ruby rubygems ruby-devel gcc rpm-build
```

* 配置RubyGems源

```bash
# 移除默认源
gem sources --remove https://rubygems.org/
# 配置阿里源
gem sources -a https://mirrors.aliyun.com/rubygems/
```

* 安装 fpm

```bash
gem install fpm
```

