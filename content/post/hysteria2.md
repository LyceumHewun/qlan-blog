---
title: "Hysteria2 部署笔记"
date: 2025-12-30T14:30:00+08:00
draft: false
tags: ["Hysteria2", "Networking", "Linux", "UDP", "Mihomo"]
categories: ["Backend"]
author: "谦兰君"
description: "Hysteria2 是一款基于 QUIC 的双边加速传输工具。相比于第一代，它完全重写了协议，并引入了更强的抗探测能力和 HTTP 认证模式。本文记录如何利用其“端口跳跃”特性来突破 UDP 限制"
---

## 1. 安装与初始化

Hysteria2 提供了一键安装脚本，支持大部分 Linux 发行版

```bash
# 执行官方安装脚本
bash <(curl -fsSL [https://get.hy2.sh/](https://get.hy2.sh/))
```

安装完成后，主程序位于 `/usr/local/bin/hysteria`，配置文件默认位于 `/etc/hysteria/config.yaml`

------

## 2. 服务端配置 (YAML)

以下配置启用了 **ACME 自动证书申请**、**HTTP 后端认证**以及 **反向代理伪装**


```yaml
# /etc/hysteria/config.yaml
listen: :443 

acme:
  domains:
    - your.domain.net  # 替换为你的解析好的域名
  email: your@email.com # 接收证书通知的邮箱

auth:
  # 使用 HTTP 认证模式，适合对接自己的用户管理系统
  http:
    url: [http://your.backend.com/auth](http://your.backend.com/auth) 
    insecure: false 

masquerade: 
  type: proxy
  proxy:
    url: [https://news.ycombinator.com/](https://news.ycombinator.com/) # 流量探测时伪装的目标
    rewriteHost: true
```

------

## 3. 核心黑科技：端口跳跃 (Port Hopping)

由于部分运营商会对单一 UDP 端口进行限速或断流，通过 `iptables` 实现多端口转发（端口跳跃）可以显著提升稳定性

### 3.1 开启转发规则

以下命令将服务器 `20000` 到 `50000` 端口的 UDP 流量全部转发至真正的监听端口 `443`：


```bash
iptables -t nat -A PREROUTING -i eth0 -p udp --dport 20000:50000 -j REDIRECT --to-ports 443
ip6tables -t nat -A PREROUTING -i eth0 -p udp --dport 20000:50000 -j REDIRECT --to-ports 443
```

> **注意**：请确保你的云服务商控制台（防火墙/安全组）已放行 `20000:50000` 的 UDP 端口

------

## 4. 客户端配置 (Clash Meta / Mihomo)

在客户端使用 `ports` 参数开启跳跃功能，`hop-interval` 控制跳跃频率（秒）


```yaml
proxies:
  - name: "Hysteria2-Node"
    type: hysteria2
    server: your.domain.net
    # 端口跳跃范围，必须与服务端 iptables 规则一致
    ports: "20000-50000"
    password: "你的密码或认证Token"
    hop-interval: 10
    skip-cert-verify: false
    up: "100"    # 上行带宽 (Mbps)
    down: "500"  # 下行带宽 (Mbps)
    udp: true
```

------

## 5. 管理与维护

使用 `systemd` 管理 Hysteria2 服务：

- **启动服务**：`systemctl start hysteria-server.service`
- **开机自启**：`systemctl enable hysteria-server.service`
- **查看日志**：`journalctl -u hysteria-server.service -f`

## 6. 避坑指南

1. **带宽限制**：Hysteria2 的发包机制较为激进，建议在客户端准确填写 `up` 和 `down` 参数，否则可能导致网络拥塞或被运营商封锁
2. **权限问题**：如果使用 `iptables` 转发，确保系统内核支持 `nat` 表
3. **证书申请**：ACME 模式下，务必确保服务器的 80 端口没有被其他程序（如 Nginx）独占，除非改用 DNS 验证
