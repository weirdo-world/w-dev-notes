# Ubuntu 系统操作
## 文件编辑
```shell
# 编辑文件
vi /tmp/test.txt
#拷贝行
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
````

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

````

## 修改端口号
```shell
# 找的里面的 Port 修改自己的端口
vi /etc/ssh/sshd_config

# 重启服务
systemctl restart sshd
```
## ufw防火墙
```shell
# 查看状态
ufw status
# 开启防火墙
ufw enable
# 关闭防火墙
ufw disable
# 添加端口
ufw allow 80/tcp
# 删除端口
ufw delete allow 80/tcp

```

