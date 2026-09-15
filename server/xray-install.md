# Xray 安装与配置（VLESS + REALITY）

## 方式一：一键安装（官方 release 脚本）

Xray 官方提供安装脚本，一条命令完成安装与 systemd 服务注册；密钥、UUID、shortId 需要自己生成并填入配置。

```shell
# 安装（官方脚本，自动识别架构并注册 xray.service）
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install

# 生成 UUID（记下输出，填到 config.json 的 clients[].id）
xray uuid
# 示例输出：<你的UUID>

# 生成 REALITY 密钥对（PrivateKey 填服务端，PublicKey 填客户端）
xray x25519
# 示例输出：
# PrivateKey: <你的PrivateKey>
# Password (PublicKey): <你的PublicKey>
# Hash32: <你的Hash32>

# 生成 shortId
openssl rand -hex 8
# 示例输出：<你的shortId>
```

## 方式二：配置 /usr/local/etc/xray/config.json

> 说明：Xray 自身解析器支持 `//` 注释（非标准 JSON，其他工具解析会报错）；生产环境建议把端口改为 443。
> 下面所有 `<...>` 占位符替换为「方式一」中生成的你自己的值，**不要照抄示例**。

```json
{
  "log": {
    "loglevel": "warning"
  },
  "dns": {
    "servers": [
      {
        "address": "8.8.8.8",
        "domains": ["gstatic.com","google.com","googleapis.com","googleusercontent.com"]
      },
      "1.1.1.1"
    ]
  },
  "inbounds": [
    {
      "port": 80, //端口
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "<你的UUID>",
            "flow": "xtls-rprx-vision",
            "level": 0
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "dest": "www.microsoft.com:443",
          "serverNames": ["www.microsoft.com", "www.bing.com"],
          "privateKey": "<你的PrivateKey>", // 对应 PublicKey：<你的PublicKey>
          "shortIds": ["<你的shortId>"], // 自己的shortId
          "fingerprint": "chrome"
        }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {
        "domainStrategy": "UseIP"
      },
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "tag": "blocked"
    }
  ],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      },
      {
        "type": "field",
        "domain": ["geosite:category-ads"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```

## 校验并重启

```shell
# 重启前先校验配置，输出 Configuration OK 再继续
xray -test -config /usr/local/etc/xray/config.json

# 如开启 ufw，先放行监听端口（示例为 80；改成 443 则放行 443）
ufw allow 80/tcp

systemctl restart xray.service
systemctl status xray.service
```
