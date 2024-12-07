# 远程桌面二次验证

## 安装multiOTP软件

{% embed url="https://download.multiotp.net/credential-provider/" %}

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>



## 配置



```
test 为Windows用户名
C:\Program Files\multiOTP
用户初始化二次验证信息
multiotp.exe -fastcreatenopin test
生成二次验证密钥二维码
multiotp.exe -qrcode test qrcode-test.png

```
