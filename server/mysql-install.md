# Ubuntu 安装 MySQL

适用 Ubuntu 22.04 / 24.04，apt 安装 MySQL 8.x。安装完成后可继续做端口调整、参数优化与远程账号配置。

## 方式一：一键安装脚本

自包含脚本，完成系统更新、安装与服务状态校验，可重复执行：

```shell
#!/usr/bin/env bash
# Ubuntu 22.04/24.04 一键安装 MySQL
set -e

echo "[1/3] 更新系统..."
sudo apt update && sudo apt upgrade -y

echo "[2/3] 安装 MySQL Server..."
sudo apt install -y mysql-server

echo "[3/3] 校验版本与服务状态..."
mysql --version
sudo systemctl status mysql.service --no-pager

echo '完成。请继续手动执行 sudo mysql_secure_installation 做安全加固（交互式，见「方式二」第 2 步）。'
```

```shell
vi install-mysql.sh   # 粘贴上面的内容
chmod +x install-mysql.sh
./install-mysql.sh
```

> 注意：`mysql_secure_installation` 是交互式问答，无法脚本化，必须手动执行，见「方式二：手动逐步安装」第 2 步。

## 方式二：手动逐步安装

### 第 1 步：更新系统并安装

```shell
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装 MySQL
sudo apt install mysql-server -y

# 查看 MySQL 版本验证安装
mysql --version
```

### 第 2 步：运行安全配置脚本

```shell
sudo mysql_secure_installation
```

#### 2.1 密码验证组件

```text
VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?
Press y|Y for Yes, any other key for No:

选择： y

说明： 启用密码强度验证组件，强制使用安全密码。
```

#### 2.2 密码验证等级

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

#### 2.3 移除匿名用户

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

#### 2.4 禁止 root 远程登录

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

#### 2.5 移除测试数据库

```text
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.

Remove test database and access to it? (Press y|Y for Yes, any other key for No):
选择： y

结果：

- Dropping test database...
Success.

- Removing privileges on test database...
Success.

说明： 删除默认的 test 数据库及其访问权限。
```

#### 2.6 重新加载权限表

```text
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No):
选择： y

结果： Success.

说明： 刷新权限表，使所有更改立即生效。
```

## 修改端口与监听地址

配置文件 `/etc/mysql/mysql.conf.d/mysqld.cnf`：

```ini
[mysqld]
port = 3306
bind-address = 0.0.0.0
```

> 注意：`bind-address = 0.0.0.0` 表示监听所有网卡，必须搭配防火墙白名单和强密码使用。

## 参数优化

新建 `/etc/mysql/conf.d/mysql-tuning.cnf`（`conf.d` 下的文件会被自动加载，便于与发行版默认配置分离）：

```ini
[mysqld]

# 基础连接
port = 3306
max_connections = 50

# 内存配置 (重点!)
innodb_buffer_pool_size = 256M      # 物理内存的 25%
innodb_log_buffer_size = 8M
innodb_log_file_size = 64M          # 8.0.30 起废弃，改用 innodb_redo_log_capacity = 128M；8.4+ 已移除

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

## 重启 MySQL

```shell
sudo systemctl restart mysql.service
```

## 创建远程登录账号

```sql
-- 登录（auth_socket 认证下无需密码）
-- sudo mysql
CREATE USER 'root'@'%' IDENTIFIED BY '替换为你的强密码';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

```shell
# 如开启 ufw，需放行 3306 端口远程才能连上
sudo ufw allow 3306/tcp
```

> 更安全的做法：为业务创建专用账号并只授权指定数据库，例如
> `CREATE USER 'app'@'%' IDENTIFIED BY '强密码'; GRANT ALL PRIVILEGES ON appdb.* TO 'app'@'%';`
