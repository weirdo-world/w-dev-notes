## Ubuntu 安装MySQL
```shell
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装MySQL
sudo apt install mysql-server -y

# 查看 MySQL 版本验证安装
mysql --version

```
## 运行安全配置脚本
```shell
sudo mysql_secure_installation
```

1. 密码验证组件
```text
VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?
Press y|Y for Yes, any other key for No:

选择： y

说明： 启用密码强度验证组件，强制使用安全密码。

 ```
2. 密码验证等级
```text
There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG:

选择： 2 (STRONG)

说明：

0 (LOW)：仅要求密码长度 ≥ 8

1 (MEDIUM)：长度 ≥ 8，包含数字、大小写字母、特殊字符

2 (STRONG)：在 MEDIUM 基础上增加字典文件检查

注意： 由于使用 auth_socket 认证插件，本次未提示设置 root 密码，需后续手动设置。
```
3. 移除匿名用户
```text
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No):
选择： y

结果： Success.

说明： 删除匿名用户，防止未授权访问。
```
4. 禁止 root 远程登录
```text
Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No):
选择： n

结果： ... skipping.

说明：

y：禁止 root 远程登录（更安全，推荐生产环境）

n：允许 root 远程登录（当前选择）

注意： 允许 root 远程登录存在安全风险，建议后续创建专用用户进行远程管理。
```
5. 移除测试数据库
```text
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.

Remove test database and access to it? (Press y|Y for Yes, any other key for No):
选择： y

结果：

text
- Dropping test database...
Success.

- Removing privileges on test database...
Success.
说明： 删除默认的 test 数据库及其访问权限。
```
6. 重新加载权限表
```text
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No):
选择： y

结果： Success.

说明： 刷新权限表，使所有更改立即生效。
```
## 修改端口 /etc/mysql/mysql.conf.d/mysqld.cnf
```shell
[mysqld]
port = 3306
bind-address = 0.0.0.0
```
## 配置优化 创建 /etc/mysql/conf.d/test.cnf
```shell
[mysqld]

# 基础连接
port = 3306
max_connections = 50

# 内存配置 (重点!)
innodb_buffer_pool_size = 256M      # 物理内存的 25%
innodb_log_buffer_size = 8M
innodb_log_file_size = 64M

key_buffer_size = 32M                # MyISAM 索引缓存

# 临时表
tmp_table_size = 16M
max_heap_table_size = 16M

# 排序缓冲区
sort_buffer_size = 1M
join_buffer_size = 1M
read_buffer_size = 512K
read_rnd_buffer_size = 512K

# 表缓存
table_open_cache = 200
table_definition_cache = 200

# 线程
thread_cache_size = 8

# InnoDB 配置
innodb_flush_log_at_trx_commit = 2   # 提升写入性能
innodb_flush_method = O_DIRECT

# 慢查询日志 (调试用)
slow_query_log = 0
long_query_time = 2

# 字符集
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

```

## 重启MySQL
```shell
systemctl restart mysql.service
```
## 创建远程登录账号
```shell
# 登录
sudo mysql
# 创建远程用户
CREATE USER 'root'@'%' IDENTIFIED BY 'aB9bDZ&AjC';
# 授权
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
# 刷新权限
FLUSH PRIVILEGES;
# 退出
exit
```

