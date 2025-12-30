---
title: " VLESS + Reality + Vision 部署笔记"
date: 2025-12-30T12:00:00+08:00
draft: false
tags: ["Xray", "VLESS", "Reality", "Network", "VPS"]
categories: ["Backend"]
author: "谦兰君"
description: "本文记录了如何从零开始在 Linux 服务器上部署 `VLESS + Reality + Vision` 组合。该方案通过 Reality 技术消除 TLS 指纹特征，并配合 Vision 流控实现高性能、高隐蔽性的访问"
---

## 1. 准备工作

### 1.1 选择目标域名
Reality 需要一个目标域名作为伪装。理想的选择是：国外知名、支持 TLSv1.3、且在国内访问延迟较低的网站
* **辅助工具**：[DNSlytics](https://search.dnslytics.com/) (用于查询同 IP 下的优质域名)

### 1.2 系统网络优化
在部署前，建议开启 BBRv3 或进行 TCP 优化以提升吞吐量

**安装 BBRv3：**
```bash
bash <(curl -l -s [https://raw.githubusercontent.com/byJoey/Actions-bbr-v3/refs/heads/main/install.sh](https://raw.githubusercontent.com/byJoey/Actions-bbr-v3/refs/heads/main/install.sh))
```

**综合 TCP 优化脚本：**

```bash
wget -O tcpx.sh "[https://github.com/ylx2016/Linux-NetSpeed/raw/master/tcpx.sh](https://github.com/ylx2016/Linux-NetSpeed/raw/master/tcpx.sh)" && chmod +x tcpx.sh && ./tcpx.sh
```

---

## 2. 安装与环境配置

### 2.1 安装 Xray 核心

使用官方脚本一键安装：

```bash
bash -c "$(curl -L [https://github.com/XTLS/Xray-install/raw/main/install-release.sh](https://github.com/XTLS/Xray-install/raw/main/install-release.sh))" @ install
```

### 2.2 生成必要参数

在配置前，需要生成 UUID 以及 Reality 专用的密钥对

* **生成 UUID**：
```bash
xray uuid
# 输出示例：2233ebed-68b0-4606-a241-1be5f8ad4668
```


* **生成 x25519 密钥对**：
```bash
xray x25519
# PrivateKey: aIRW_Eh8-n8JEmkHFRxQeHnBipHrxt6OrIbAeUVr12s
# PublicKey:  YLr6CDT0jCxaxZaDypHnOZzB4D83MWLwR06nSWykzBI
```



---

## 3. 服务端配置

编辑配置文件：`/usr/local/etc/xray/config.json`

> **注意**：Reality 必须监听 443 端口以实现完美伪装。

```json
{
  "log": {
    "loglevel": "error"
  },
  "api": {
    "tag": "api",
    "services": ["HandlerService", "LoggerService", "StatsService", "RoutingService"]
  },
  "inbounds": [
    {
      "listen": "0.0.0.0",
      "port": 10086,
      "protocol": "dokodemo-door",
      "settings": { "address": "127.0.0.1" },
      "tag": "api"
    },
    {
      "port": 443,
      "tag": "vless-reality",
      "protocol": "vless",
      "settings": {
        "clients": [],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "show": false,
          "dest": "www.lovelive-anime.jp:443", // 目标域名及端口
          "xver": 0,
          "serverNames": ["www.lovelive-anime.jp"], // 客户端连接时使用的域名
          "privateKey": "aIRW_Eh8-n8JEmkHFRxQeHnBipHrxt6OrIbAeUVr12s", // 填入生成的 PrivateKey
          "shortIds": ["1d582b6c"] // 随机 1-8 位十六进制字符
        }
      }
    }
  ],
  "outbounds": [
    { "protocol": "freedom", "tag": "direct" },
    { "protocol": "blackhole", "tag": "block" }
  ],
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      { "inboundTag": ["api"], "sourceIP": ["0.0.0.0/0"], "outboundTag": "api", "type": "field" },
      { "type": "field", "ip": ["geoip:private"], "outboundTag": "block" },
      { "type": "field", "protocol": ["bittorrent"], "outboundTag": "block" }
    ]
  }
}
```

配置完成后，重启 Xray：

```bash
systemctl restart xray
systemctl status xray
```

---

## 4. 客户端配置 (Clash/Mihomo)

以 Clash Meta (Mihomo) 核心为例，在 `proxies` 章节添加以下配置：

```yaml
proxies:
  - name: "Reality-Vision-Node"
    type: vless
    server: 142.171.47.230      # 你的 VPS IP
    port: 443
    uuid: 2233ebed-68b0-4606-a241-1be5f8ad4668
    tls: true
    udp: true
    flow: xtls-rprx-vision
    servername: www.lovelive-anime.jp # 须与服务端 serverNames 一致
    reality-opts:
      public-key: YLr6CDT0jCxaxZaDypHnOZzB4D83MWLwR06nSWykzBI # 填入生成的 PublicKey
      short-id: 1d582b6c # 须与服务端 shortIds 一致
    client-fingerprint: chrome # 模拟浏览器指纹
```

---

## 5. 参考资源

* [Xray 官方文档](https://xtls.github.io/)
* [Remnawave XTLS SDK](https://github.com/remnawave/xtls-sdk)
