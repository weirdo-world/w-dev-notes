# Ubuntu 系统操作

## 文件编辑（vi）

```shell
# 编辑文件
vi /tmp/test.txt

# 拷贝行
yy

# 粘贴行
p

# 删除所有内容
:%d

# 删除当前行
dd

# 删除当前行到文件末尾
dG

# 退出保存文件
:wq
```

## 文件操作

```shell
# 创建文件
touch /tmp/test.txt

# 创建目录
mkdir /tmp/test

# 删除文件
rm /tmp/test.txt

# 删除目录
rm -r /tmp/test

# 移动文件
mv /tmp/test.txt /tmp/test.txt.bak

# 复制文件
cp /tmp/test.txt /tmp/test.txt.bak
```

## 修改 SSH 端口号

```shell
# 找到文件中的 Port 修改为自己的端口（示例 2222）
sudo vi /etc/ssh/sshd_config

# 必须先放行新端口，再重启服务，否则当前 SSH 会断开且无法重新连接
sudo ufw allow 2222/tcp

# Ubuntu 的服务名是 ssh（CentOS/RHEL 才是 sshd）
sudo systemctl restart ssh
```

## ufw 防火墙

```shell
# 查看状态
sudo ufw status

# 开启前必须先放行 SSH 端口，否则会把自己关在外面
sudo ufw allow OpenSSH

# 开启防火墙
sudo ufw enable

# 关闭防火墙
sudo ufw disable

# 添加端口
sudo ufw allow 80/tcp

# 删除端口
sudo ufw delete allow 80/tcp
```
