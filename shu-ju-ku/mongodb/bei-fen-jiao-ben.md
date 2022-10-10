# 备份脚本

```bash
#!/bin/bash
set -e

#### DB信息 ####

# 数据库地址
db_host=localhost
# 数据库端口
db_port=27017
# 数据名称
db_name=test
# 数据库用户名
db_user=test
# 数据库密码
db_pass=test
# 登录认证DB
db_auth_db=admin

###### 备份信息 ######

# 保留周期(天)
retention_period=180
# 备份目录
backup_path=/data/backup/mongodump
# 备份文件路径
backup_file=$backup_path/$db_name-$(date "+%Y%m%d%H%M%S").gz
# 日志文件
log_file=$backup_path/backup.log

###############


# 如果备份目录不存在,则创建备份目录
if [ ! -d $backup_path ]; then
    mkdir -p $backup_path
fi

echo "---------------------------" >> $log_file
echo $(date "+%Y-%m-%d %H:%M:%S") "开始备份数据库" >> $log_file

# 如果备份数据库文件存在,则删除
if [ -f $backup_file ]; then
    rm -f $backup_file
fi

echo "数据库备份路径: " $backup_file >> $log_file

######## 备份数据库 ###########

# mongodb
mongodump --host=$db_host --port=$db_port --username=$db_user --password=$db_pass --authenticationDatabase=$db_auth_db --db=$db_name --gzip --archive=$backup_file

echo $(date "+%Y-%m-%d %H:%M:%S") "备份数据库完成" >> $log_file


# 删除过期文件
echo $(date "+%Y-%m-%d %H:%M:%S") "开始自动删除过期备份" >> $log_file

find $backup_path -mtime "+$retention_period" -name "$db_name-*" >> $log_file
find $backup_path -mtime "+$retention_period" -name "$db_name-*" -exec rm -rf {} \;

echo $(date "+%Y-%m-%d %H:%M:%S") "自动删除过期备份完成" >> $log_file
echo "---------------------------" >> $log_file


```
