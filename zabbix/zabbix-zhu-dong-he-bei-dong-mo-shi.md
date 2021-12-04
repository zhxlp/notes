# Zabbix 主动和被动模式

- 被动模式

  Zabbix server 主动连接 agent 获取数据

  agent 端开放端口

- 主动模式

  Zabbix agent 主动向 server 发送数据

  server 端开发端口

## 被动模式配置

- 修改`zabbix_agentd.conf`配置文件

  ```properties
  # zabbix允许那些地址访问
  Server=
  # 启动agent监听
  StartAgents=3
  # agent主机名称,与zabbix前端添加主机名时的名称一致
  Hostname=
  ```

- 前端页面添加主机

  【配置】【主机】【创建主机】

  **主机名称**：与 agent 配置文件中 Hostname 一致

  **agent 代理程序的接口**：agent 的地址，并且开放端口

## 主动模式配置

- 修改`zabbix_agentd.conf`配置文件

  ```properties
  # zabbix允许那些地址访问
  Server=127.0.0.1
  # 关闭agent监听
  StartAgents=0
  # 向那些server发送信息
  ServerActive=
  # agent主机名称,与zabbix前端添加主机名时的名称一致
  Hostname=
  ```

- 前端页面添加主机

  【配置】【主机】【创建主机】

  **主机名称**：与 agent 配置文件中 Hostname 一致

  **agent 代理程序的接口**：使用模式 127.0.0.1,不更改
