# Clash 故障转移配置完全指南

[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/CG-spring/clash-fallback-config.svg?style=flat-square)](https://github.com/CG-spring/clash-fallback-config/stargazers)

> Clash Proxy Group 高可用配置 - 自动故障转移、负载均衡、延迟测试
> 
> 持续更新中 | 最后更新: 2026-04-09

**中文** | **[English](README_EN.md)**

---

## 目录

- [什么是故障转移](#什么是故障转移)
- [Proxy Group 类型](#proxy-group-类型)
- [故障转移配置](#故障转移配置)
- [负载均衡配置](#负载均衡配置)
- [健康检查配置](#健康检查配置)
- [最佳实践](#最佳实践)

---

## 什么是故障转移

故障转移（Failover）是指当主节点不可用时，自动切换到备用节点的机制。在 Clash 中，通过 `proxy-groups` 实现：

```
主节点 (低延迟) 
    ↓ 检测失败
备用节点 1 → 备用节点 2 → 备用节点 3
    ↓ 全部失败
直连/拒绝
```

**优势：**
- 提高连接稳定性
- 减少手动切换
- 自动选择最优节点

---

## Proxy Group 类型

### 1. select - 手动选择

```yaml
proxy-groups:
  - name: "手动选择"
    type: select
    proxies:
      - 香港节点1
      - 香港节点2
      - 日本节点
      - 自动选择
      - DIRECT
```

### 2. url-test - 自动测速选择

```yaml
proxy-groups:
  - name: "自动选择"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300  # 每 300 秒测速一次
    tolerance: 50  # 延迟差 50ms 内不切换
    proxies:
      - 香港节点1
      - 香港节点2
      - 日本节点
```

### 3. fallback - 故障转移

```yaml
proxy-groups:
  - name: "故障转移"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 300
    proxies:
      - 香港节点1  # 优先使用
      - 香港节点2  # 节点1失败时切换
      - 日本节点   # 节点2失败时切换
      - DIRECT     # 全部失败直连
```

### 4. load-balance - 负载均衡

```yaml
proxy-groups:
  - name: "负载均衡"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 300
    strategy: round-robin  # 或 consistent-hashing
    proxies:
      - 香港节点1
      - 香港节点2
      - 日本节点
```

---

## 故障转移配置

### 基础配置模板

```yaml
# Clash 配置文件
proxies:
  - name: "香港01"
    type: ss
    server: hk1.example.com
    port: 443
    cipher: aes-256-gcm
    password: "password"
    
  - name: "香港02"
    type: ss
    server: hk2.example.com
    port: 443
    cipher: aes-256-gcm
    password: "password"
    
  - name: "日本01"
    type: ss
    server: jp1.example.com
    port: 443
    cipher: aes-256-gcm
    password: "password"

proxy-groups:
  # 主组 - 故障转移
  - name: "PROXY"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    lazy: true  # 无请求时不检测
    proxies:
      - 香港最优
      - 日本最优
      - 备用组
      - DIRECT

  # 香港组 - 自动选速
  - name: "香港最优"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    proxies:
      - 香港01
      - 香港02

  # 日本组 - 自动选速
  - name: "日本最优"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    proxies:
      - 日本01

  # 备用组 - 故障转移
  - name: "备用组"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 300
    proxies:
      - 日本01
      - DIRECT

rules:
  - GEOIP,CN,DIRECT
  - MATCH,PROXY
```

### 进阶配置 - 多层故障转移

```yaml
proxy-groups:
  # 第一层：按地区分组
  - name: "香港精选"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - 香港01
      - 香港02
      - 香港03

  - name: "日本精选"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - 日本01
      - 日本02

  - name: "新加坡精选"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - 新加坡01
      - 新加坡02

  # 第二层：地区故障转移
  - name: "亚洲组"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - 香港精选
      - 日本精选
      - 新加坡精选

  # 第三层：全球故障转移
  - name: "PROXY"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - 亚洲组
      - 美国组
      - 欧洲组
      - DIRECT
```

---

## 负载均衡配置

### Round-Robin 模式

```yaml
proxy-groups:
  - name: "负载均衡"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 180
    strategy: round-robin  # 轮询模式
    proxies:
      - 节点1
      - 节点2
      - 节点3
```

**特点：** 请求依次分配到不同节点，适合大流量场景。

### Consistent-Hashing 模式

```yaml
proxy-groups:
  - name: "一致性哈希"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 180
    strategy: consistent-hashing
    proxies:
      - 节点1
      - 节点2
      - 节点3
```

**特点：** 同一目标始终使用同一节点，适合需要会话保持的场景。

---

## 健康检查配置

### 检测 URL 选择

```yaml
# 推荐的检测 URL
url-test / fallback / load-balance:
  url: http://www.gstatic.com/generate_204
  # 或
  url: http://cp.cloudflare.com/generate_204
  # 或
  url: https://connect.rom.miui.com/generate_204
```

### 检测参数说明

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| interval | 检测间隔 | 120-300秒 |
| tolerance | 延迟容差 | 30-50ms |
| lazy | 懒加载 | true |
| timeout | 超时时间 | 5000ms |

### 完整健康检查配置

```yaml
proxy-groups:
  - name: "智能选择"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    lazy: true
    timeout: 5000
    max-fails: 3  # 连续3次失败标记为不可用
    proxies:
      - 节点1
      - 节点2
```

---

## 最佳实践

### 1. 分场景使用不同策略

```yaml
proxy-groups:
  # 流媒体 - 故障转移
  - name: "Netflix"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - Netflix专用节点1
      - Netflix专用节点2
      - PROXY

  # 聊天软件 - 低延迟优先
  - name: "Telegram"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 20
    proxies:
      - 香港01
      - 香港02
      - 日本01

  # 下载 - 负载均衡
  - name: "下载"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 300
    strategy: round-robin
    proxies:
      - 大流量节点1
      - 大流量节点2
```

### 2. 避免频繁切换

```yaml
proxy-groups:
  - name: "稳定选择"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300      # 5分钟检测一次
    tolerance: 100     # 100ms 内不切换
    lazy: true         # 无请求不检测
    proxies:
      - 节点列表
```

### 3. 配合规则使用

```yaml
rules:
  # 流媒体走专用组
  - DOMAIN-SUFFIX,netflix.com,Netflix
  - DOMAIN-SUFFIX,disneyplus.com,Disney
  
  # 聊天软件走低延迟组
  - DOMAIN-SUFFIX,telegram.org,Telegram
  - DOMAIN-SUFFIX,t.me,Telegram
  
  # 其他走主组
  - MATCH,PROXY
```

---

## 推荐资源

- [机场导航](https://nav.clashvip.net) - 优质机场推荐
- [Clash 教程](https://clash-for-windows.net) - 客户端下载
- [用户社区](https://bbs.clashhub.net) - 技术交流
- [ClashVIP](https://clashvip.net) - 规则订阅

---

## License

CC BY-NC-SA 4.0 - 仅供学习交流，禁止商用
