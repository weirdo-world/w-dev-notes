## xray安装
```shell
# 安装
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install

# 生成UUID
xray uuid
# 输出 e3b0bd1b-4845-409f-8003-6b151b799e65

# 生成 REALITY 密钥对
xray x25519
# 输出
# PrivateKey: EH76ah-Uzta22CpF-Xbtonqxbnd23RdwUCOfJI64fnA
# Password (PublicKey): hMvDBiIXbhtczVwnq5a88yExdkSa7ZT_IGYxCwwZZkU
# Hash32: VR0f1Xr8aEZB5T0JlgtKu-rbsduhcSOfGAmXv7OOr_0

# 重新生成一个 shortId
openssl rand -hex 8
# 输出 9c5d0a0f0c5c0f


```

## 配置 /usr/local/etc/xray/config.json
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
            "id": "e3b0bd1b-4845-409f-8003-6b151b799e65",
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
          "privateKey": "EH76ah-Uzta22CpF-Xbtonqxbnd23RdwUCOfJI64fnA", // hMvDBiIXbhtczVwnq5a88yExdkSa7ZT_IGYxCwwZZkU
          "shortIds": ["9c5d0a0f0c5c0f"], // 自己的shortId
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
## 重启
```shell
systemctl restart xray.service
systemctl status xray.service
```