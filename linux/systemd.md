# Systemd

## 异常关闭,自动重启

```properties
[Unit]
Description=nwct
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/bin/Device
ExecStart=/opt/bin/Device/Device
# 异常关闭，自动重启
Restart=always
# 不限次数重启
StartLimitInterval=0
# 重启间隔
RestartSec=5

[Install]
WantedBy=multi-user.target
```
