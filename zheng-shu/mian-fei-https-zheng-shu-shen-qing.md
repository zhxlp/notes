# 免费 https 证书申请

- 简介

  Let's Encrypt —— 是一个由非营利性组织 互联网安全研究小组（ISRG）提供的免费、自动化和开放的证书颁发机构（CA），简单的说，就是为网站提供免费的 SSL/TLS 证书。

  我们可以使用 ACME 客户端进行申请证书，下面将介绍使用 ACME 客户端之一的 acme.sh 进行申请

- 安装

  ```bash
  curl https://get.acme.sh | sh
  ```

  详情请参考官方文档[安装 acme.sh](https://github.com/Neilpang/acme.sh/wiki/说明)

- 手动 dns 模式申请证书

  参考[手动 DNS 模式](https://github.com/Neilpang/acme.sh/wiki/DNS-manual-mode)

  1.获取 TXT 解析配置信息

  ```bash
  acme.sh --issue -d example.com --dns \
   --yes-I-know-dns-manual-mode-enough-go-ahead-please
  ```

  2.在域名控制台按要求配置 TXT 解析

  3.使用`--renew`命令验证并获取证书

  注：只有一次验证机会，请确保 TXT 解析全球生效(一般等待几个小时)[TXT 解析查询](https://mxtoolbox.com/TXTLookup.aspx)

  ```bash
  acme.sh --renew -d example.com --dns \
   --yes-I-know-dns-manual-mode-enough-go-ahead-please
  ```
