# Rdp File Setting

```properties
# 远程计算机的名称或IP地址
full address:s:value

# 远程服务端口
server port:i:value

# 远程用户名
username:s:value

# 远程用户密码
# ("MySuperSecretPassword!" | ConvertTo-SecureString -AsPlainText -Force) | ConvertFrom-SecureString;
password 51:b:value


# 备用地址：主机名或者IP地址
alternate full address:s:value
# 确定在连接时启动远程应用程序的路径或别名,
alternate shell:s:value
# 是否启用了音频输入/输出重定向 0:禁用 1:启用
audiocapturemode:i:value
# 确定本地机器还是远程机器播放音频 0:本地计算机播放 1:远程计算机上播放 2:不播放声音
audiomode:i:value
# 定义服务器身份验证级别设置
# 0:如果服务器身份验证失败，则在没有警告的情况下连接到计算机
# 1:如果服务器身份验证失败，请不要建立连接
# 2:如果服务器身份验证失败，显示警告并允许我连接或拒绝连接
# 3:未指定身份验证要求
authentication level:i:value

# 断开连接时客户端计算机是否将自动尝试重新连接到远程计算机
# 0:客户端计算机不会自动尝试重新连接
# 1:客户端计算机会自动尝试重新连接
autoreconnection enabled:i:value

# 确定是否启用自动网络类型检测
# 0:禁用自动网络类型检测
# 1:启用自动网络类型检测
bandwidthautodetect:i:value


camerastoredirect:s:value

# 确定批量压缩是否通过RDP传输到本地计算机时启用
# 0:禁用RDP批量压缩
# 1:启用RDP批量压缩
compression:i:value

# 通过一组预定义选项指定远程会话桌面的尺寸。如果指定了desktopheight或desktopwidth，则将覆盖此设置
# 0: 640×480
# 1: 800×600
# 2: 1024×768
# 3: 1280×1024
# 4: 1600×1200
desktop size id:i:value

# 远程计算机上的分辨率高度
desktopheight:i:value

# 远程计算机上的分辨率宽度
desktopwidth:i:value

# 确定客户端计算机上的哪些设备将被重定向并在远程会话中可用
#
#
devicestoredirect:s:value

# 确定启动RemoteApp或桌面时，远程桌面客户端是重新连接到任何现有的打开连接还是发起新连
# 0：重新连接到任何现有会话
# 1：启动新连接
disableconnectionsharing:i:value

# 指定将用于登录到远程计算机的用户帐户所在的域的名称
domain:s:value

# 确定客户端计算机上的哪些本地磁盘驱动器将被重定向并在远程会话中可用
#
drivestoredirect:s:value

# 确定RDP是否将使用凭据安全支持提供程序（CredSSP）进行身份验证
# 0：即使操作系统支持CredSSP，RDP也不会使用CredSSP
# 1：如果操作系统支持CredSSP，则RDP将使用CredSSP
enablecredsspsupport:i:value

# 启用或禁用重定向视频的编码
# 0：禁用重定向视频的编码
# 1：启用重定向视频的编码
encode redirected video capture:i:value



# 指定或检索RD网关身份验证方法
# 0：要求输入密码
# 1：使用智能卡
# 4：允许用户以后选择
gatewaycredentialssource:i:value

# 指定RD网关主机名
gatewayhostname:s:value

# 指定是否使用默认的RD网关设置
# 0：使用管理员指定的默认配置文件模式
# 1：使用用户指定的显式设置
gatewayprofileusagemethod:i:value

# 指定何时使用RD网关服务器
# 0：不使用RD网关服务器
# 1：始终使用RD网关服务器
# 2：如果无法直接连接到RD会话主机，则使用RD网关服务器
# 3：使用默认的RD网关服务器设置
# 4：请勿使用RD网关，将本地地址绕过服务器
gatewayusagemethod:i:value

# 确定是否使用自动网络带宽检测。需要设置选项带宽自动检测并与连接类型7相关联
# 0：不使用自动网络带宽检测
# 1：使用自动网络带宽检测
networkautodetect:i:value

# 确定是否将用户的凭据保存并用于RD网关和远程计算机
# 0：远程会话将不使用相同的凭据
# 1：远程会话将使用相同的凭据
promptcredentialonce:i:value

# 确定是否启用剪贴板重定向
# 0：远程会话中本地计算机上的剪贴板不可用
# 1：远程会话中本地计算机上的剪贴板不可用
redirectclipboard:i:value

# 控制编码视频的质量
# 0：高压缩视频。运动较多时，质量可能会下降
# 1：中压缩
# 2：高图像质量的低压缩视频
redirected video capture encoding quality:i:value

# 确定当您使用远程桌面连接连接到远程计算机时，客户端计算机上配置的打印机是否将被重定向并在远程会话中可用。
# 0：本地计算机上的打印机在远程会话中不可用
# 1：本地计算机上的打印机在远程会话中不可用
redirectprinters:i:value

# 确定当您连接到远程计算机时，客户端计算机上的智能卡设备是否将被重定向并在远程会话中可用
# 0：本地计算机上的智能卡设备在远程会话中不可用
# 1：本地计算机上的智能卡设备在远程会话中不可用
redirectsmartcards:i:value

# RemoteApp的可选命令行参数
remoteapplicationcmdline:s:value

# 确定应在本地还是远程扩展RemoteApp命令行参数中包含的环境变量
# 0：应将环境变量扩展为本地计算机的值
# 1：应将远程计算机上的环境变量扩展为远程计算机的值
remoteapplicationexpandcmdline:i:value

# 确定应在本地还是远程扩展RemoteApp工作目录参数中包含的环境变量
# 0：应将环境变量扩展为本地计算机的值
# 1：应将远程计算机上的环境变量扩展为远程计算机的值
# 通过Shell工作目录参数指定RemoteApp工作目录
remoteapplicationexpandworkingdir

# 指定要由RemoteApp在远程计算机上打开的文件。要打开本地文件，还必须为源驱动器启用驱动器重定向
remoteapplicationfile:s:value

# 指定启动RemoteApp时要在客户端UI中显示的图标文件。如果未指定文件名，则客户端将使用标准的远程桌面图标。仅支持“ .ico”文件
remoteapplicationicon:s:value

# 确定是否将RemoteApp连接作为RemoteApp会话启动
# 0：不启动RemoteApp会话
# 1：启动RemoteApp会话
remoteapplicationmode:i:value

# 启动RemoteApp时，在客户端界面中指定RemoteApp的名称
remoteapplicationname:s:value

# 指定RemoteApp的别名或可执行文件名称
remoteapplicationprogram:s:value

# 确定当您使用远程桌面连接连接到远程计算机时，远程会话窗口是否全屏显示
# 1：远程会话将出现在窗口中
# 2：远程会话将以全屏显示
screen mode id:i:value

# 确定客户端计算机是否可以缩放远程计算机上的内容以适合客户端计算机的窗口大小
# 0：调整大小时客户端窗口显示不会缩放
# 1：调整大小时客户端窗口显示将缩放
smart sizing:i:value

# 通过使用远程桌面连接连接到远程计算机时，配置多个监视器支持
# 0：不启用多显示器支持
# 1：启用多显示器支持
use multimon:i:value

# 指定将用于登录到远程计算机的用户帐户的名称
username:s:value

# 密码
# ("MySuperSecretPassword!" | ConvertTo-SecureString -AsPlainText -Force) | ConvertFrom-SecureString;
password 51:b:value

# 确定远程桌面连接是否将使用RDP有效的多媒体流进行视频播放
# 0：请勿将RDP高效的多媒体流用于视频播放
# 1：尽可能将RDP高效的多媒体流用于视频播放
videoplaybackmode:i:value

# 定义与包含此设置的RDP文件关联的RemoteApp和桌面ID
workspaceid:s:value
```
