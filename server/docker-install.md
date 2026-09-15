# Ubuntu 安装 Docker

适用 Ubuntu 22.04 / 24.04，安装内容：Docker Engine、CLI、containerd、Buildx、Compose 插件。

## 方式一：一键安装脚本

自包含脚本，使用 Docker 官方 apt 源，可重复执行：

```shell
#!/usr/bin/env bash
# Ubuntu 22.04/24.04 一键安装 Docker
set -e

echo "[1/5] 清理旧版本..."
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt-get remove -y "$pkg" 2>/dev/null || true
done

echo "[2/5] 安装依赖..."
sudo apt-get update
sudo apt-get install -y ca-certificates curl

echo "[3/5] 添加官方 GPG 密钥与软件源..."
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

echo "[4/5] 安装 Docker..."
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo "[5/5] 当前用户加入 docker 组..."
sudo usermod -aG docker "$USER"

echo "验证..."
sudo docker run --rm hello-world
echo "完成。重新登录（或执行 newgrp docker）后可免 sudo 使用 docker。"
```

```shell
vi install-docker.sh   # 粘贴上面的内容
chmod +x install-docker.sh
./install-docker.sh
```

国内机器访问官方源慢时，改用官方便捷脚本的阿里云镜像（一行完成）：

```shell
curl -fsSL https://get.docker.com | sudo sh -s -- --mirror Aliyun
```

## 方式二：手动逐步安装

**第 1 步：清理旧版本**

```shell
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt-get remove -y "$pkg" 2>/dev/null || true
done
```

**第 2 步：安装依赖**

```shell
sudo apt-get update
sudo apt-get install -y ca-certificates curl
```

**第 3 步：添加官方 GPG 密钥**

```shell
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

**第 4 步：添加 apt 软件源**

```shell
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

**第 5 步：安装 Docker**

```shell
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**第 6 步：验证**

```shell
# 看到 Hello from Docker! 即为成功；--rm 表示退出后自动删除测试容器
sudo docker run --rm hello-world

# 版本检查
docker --version
docker compose version
```

**第 7 步：免 sudo 使用 docker**

```shell
sudo usermod -aG docker "$USER"
newgrp docker      # 当前会话立即生效，重新登录也可
docker ps          # 不再需要 sudo
```

## 安装后配置（可选）

国内拉取镜像慢时配置镜像加速。公共加速器近年大量失效，建议使用云厂商提供的**个人加速器地址**（如阿里云容器镜像服务控制台免费获取）：

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json > /dev/null <<'EOF'
{
  "registry-mirrors": ["https://替换为你的加速器地址"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
docker info | grep -A 5 "Registry Mirrors"   # 确认生效
```

## 卸载

```shell
sudo apt-get purge -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
sudo rm -rf /var/lib/docker /var/lib/containerd /etc/docker
```
