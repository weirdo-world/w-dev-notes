# Ubuntu 卸载 MySQL（apt 安装版）

> 警告：以下操作会**彻底删除所有数据库、配置文件和账号信息，不可恢复**。

## 备份（可选但建议）

```shell
# 导出全部数据库到当前用户目录
mysqldump -u root -p --all-databases --routines --events > ~/all-databases.sql

# 或直接冷拷贝数据目录（需先停服务）
sudo systemctl stop mysql
sudo cp -a /var/lib/mysql ~/mysql-data-backup
```

## 一键卸载脚本

停止服务 → 清除软件包 → 删除数据/配置/日志 → 删除系统用户 → 验证：

```shell
#!/usr/bin/env bash
# 适用：apt 安装的 MySQL 8.x（Ubuntu 22.04/24.04，Debian 同理）
# 不适用：MySQL 官方源安装的 mysql-community-* 包（包名不同，需另行 purge）
set -euo pipefail

echo "[1/5] 停止 MySQL 服务..."
sudo systemctl stop mysql 2>/dev/null || echo "  服务未运行，跳过"

echo "[2/5] 当前已安装的 MySQL 相关包："
dpkg -l | grep -E 'mysql-(server|client|common)' || echo "  无"

echo "[3/5] 卸载软件包并清除配置（请留意 apt 列出的连带卸载清单！）..."
# 通配符可同时覆盖 8.0 / 8.4 等版本
sudo apt-get purge -y 'mysql-server*' 'mysql-client*' mysql-common
sudo apt-get autoremove -y
sudo apt-get autoclean

echo "[4/5] 删除数据、配置、日志目录..."
sudo rm -rf /etc/mysql /var/lib/mysql /var/log/mysql /var/run/mysqld

echo "[5/5] 删除 mysql 系统用户..."
sudo userdel mysql 2>/dev/null || echo "  用户不存在，跳过"

echo "验证卸载结果："
if dpkg -l | grep -qE '^ii +(mysql-server|mysql-common)'; then
  echo "⚠️ 仍有核心包残留："
  dpkg -l | grep -E '^ii +(mysql-server|mysql-common)'
  exit 1
fi
echo "✅ MySQL 已彻底卸载"
```

使用方式：

```shell
vi uninstall-mysql.sh   # 粘贴上面的内容
chmod +x uninstall-mysql.sh
./uninstall-mysql.sh
```

> 注意：`mysql-common` 被其他软件（如 `libmysqlclient21`）依赖时，apt 会连带卸载这些软件，执行前留意终端的卸载清单。
> 零风险预演（只模拟不执行）：`sudo apt-get purge -s 'mysql-server*' 'mysql-client*' mysql-common`

## 手动逐步卸载

**第 1 步：停止服务**

```shell
sudo systemctl stop mysql
```

**第 2 步：卸载软件包并清除配置**

```shell
# 先看清楚 apt 列出的连带卸载清单，确认无误再输入 y
sudo apt-get purge 'mysql-server*' 'mysql-client*' mysql-common
sudo apt-get autoremove
sudo apt-get autoclean
```

**第 3 步：删除数据、配置、日志目录**

```shell
sudo rm -rf /etc/mysql /var/lib/mysql /var/log/mysql /var/run/mysqld
```

**第 4 步：删除 mysql 系统用户**

```shell
sudo userdel mysql
```

**第 5 步：验证**

```shell
# 无输出表示服务端核心包已清除
dpkg -l | grep -E '^ii +(mysql-server|mysql-common)'
# 注意：libmysqlclient21 等是其他软件依赖的客户端库，残留属正常，不代表 MySQL 未卸载干净

# 命令不存在表示客户端已删除
mysql --version

# 数据/配置目录应不存在
ls -d /etc/mysql /var/lib/mysql 2>/dev/null
```
