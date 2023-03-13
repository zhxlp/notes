# 启用账号密码登录

## 设置管理员用户

```sh
mongo
```

```mongodb
use admin

db.createUser({
  user: 'admin',
  pwd: 'redhatQWE123',
  roles:[{
    role: 'root',
    db: 'admin'
  }]
})

show users
```

## 启用账号认证

修改 `/etc/mongod.conf`

```yaml
security:
  authorization: enabled
```

## 重启服务

