# Clash 故障转移配置指南

[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)
[![Stars](https://img.shields.io/github/stars/CG-spring/clash-fallback-config?style=flat-square)](https://github.com/CG-spring/clash-fallback-config/stargazers)

> 完整 Clash 代理组配置教程——故障转移（Failover）、负载均衡（Load Balance）与高可用（High Availability）

**中文** | **[English](README_EN.md)**

---

## 目录

- [什么是故障转移](#什么是故障转移)
- [代理组类型](#代理组类型)
- [故障转移配置](#故障转移配置)
- [负载均衡](#负载均衡)
- [健康检查](#健康检查)
- [最佳实践](#最佳实践)
- [相关资源](#相关资源)
- [许可证](#许可证)

---

## 什么是故障转移

故障转移（Failover）是指当主节点不可用时，系统自动切换到备用节点，从而保证连接不中断。在 Clash 中，这一能力完全通过 `proxy-groups`（代理组）实现：

```
主节点（低延迟）
    │  检测到故障
    ▼
备用 1  →  备用 2  →  备用 3
    │  全部失败
    ▼
DIRECT / REJECT
```

**核心价值：**

- 提升连接稳定性，节点掉线时无需手动切换
- 自动选择当前最优节点，减少卡顿
- 配合规则可分场景使用不同策略（流媒体、下载、聊天）

Clash 的 `fallback` 类型代理组正是为此设计：它按列表顺序从上到下尝试，只要当前节点可用就一直使用，一旦失败立刻切到下一个，直到列表末尾。

---

## 代理组类型

Clash（含 mihomo / Clash.Meta）提供多种代理组类型，理解它们的差异是写好配置的基础。

### 1. select —— 手动选择

最基础的代理组，用户在下拉菜单里手动挑选节点。

```yaml
proxy-groups:
  - name: "手动选择"
    type: select
    proxies:
      - 香港-01
      - 香港-02
      - 日本-01
      - 自动选择
      - DIRECT
```

适用场景：需要精确控制走哪条线路时（如某些站点对特定地区节点更友好）。

### 2. url-test —— 自动测速

每隔一段时间对所有节点发起探测，自动选用延迟最低的一个。

```yaml
proxy-groups:
  - name: "自动选择"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300        # 每 300 秒测一次
    tolerance: 50        # 延迟差超过 50ms 才切换
    proxies:
      - 香港-01
      - 香港-02
      - 日本-01
```

适用场景：日常浏览、对延迟敏感的应用（Telegram、游戏加速）。

### 3. fallback —— 故障转移

按顺序优先使用排在前面的节点，失败则顺延。

```yaml
proxy-groups:
  - name: "故障转移"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 300
    proxies:
      - 香港-01         # 优先使用
      - 香港-02         # 香港-01 失败时使用
      - 日本-01         # 再失败时使用
      - DIRECT          # 全部失败则直连
```

适用场景：稳定性优先的场景（视频通话、直播推流、长时间下载）。

### 4. load-balance —— 负载均衡

在多个节点之间分配流量，支持轮询（round-robin）与一致性哈希（consistent-hashing）。

```yaml
proxy-groups:
  - name: "负载均衡"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 300
    strategy: round-robin      # 或 consistent-hashing
    proxies:
      - 节点-01
      - 节点-02
      - 节点-03
```

适用场景：大流量下载、需要分散单一节点压力的场合。

---

## 故障转移配置

### 基础配置模板

下面是一个可直接套用的完整结构，先定义节点，再用嵌套代理组实现「区域优选 + 全局兜底」。

```yaml
# Clash 配置文件
proxies:
  - name: "HK-01"
    type: ss
    server: hk1.example.com
    port: 443
    cipher: aes-256-gcm
    password: "password"

  - name: "HK-02"
    type: ss
    server: hk2.example.com
    port: 443
    cipher: aes-256-gcm
    password: "password"

  - name: "JP-01"
    type: ss
    server: jp1.example.com
    port: 443
    cipher: aes-256-gcm
    password: "password"

proxy-groups:
  # 主组——故障转移
  - name: "PROXY"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    lazy: true
    proxies:
      - HK-最优
      - JP-最优
      - 备用
      - DIRECT

  # 香港组——url-test
  - name: "HK-最优"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    proxies:
      - HK-01
      - HK-02

  # 日本组——url-test
  - name: "JP-最优"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    proxies:
      - JP-01

  # 备用组——fallback
  - name: "备用"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 300
    proxies:
      - JP-01
      - DIRECT

rules:
  - GEOIP,CN,DIRECT
  - MATCH,PROXY
```

### 多层故障转移

当节点较多时，可以用「区域 → 大区 → 全局」三层结构，既保证速度又保证兜底：

```yaml
proxy-groups:
  # 第一层：按地区优选
  - name: "HK-优选"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - HK-01
      - HK-02
      - HK-03

  - name: "JP-优选"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - JP-01
      - JP-02

  # 第二层：地区间故障转移
  - name: "亚洲"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - HK-优选
      - JP-优选

  # 第三层：全球故障转移
  - name: "PROXY"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - 亚洲
      - 美国
      - 欧洲
      - DIRECT
```

---

## 负载均衡

### 轮询模式（round-robin）

```yaml
proxy-groups:
  - name: "轮询"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 180
    strategy: round-robin
    proxies:
      - 节点-01
      - 节点-02
      - 节点-03
```

**适用：** 高流量场景，把请求均匀分散到各节点，避免单点限速。

### 一致性哈希模式（consistent-hashing）

```yaml
proxy-groups:
  - name: "一致性哈希"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 180
    strategy: consistent-hashing
    proxies:
      - 节点-01
      - 节点-02
      - 节点-03
```

**适用：** 需要会话保持的场景（同一目标始终走同一节点），例如登录态、长连接。

---

## 健康检查

代理组能否正确切换，取决于健康检查参数是否合理。

### 探测 URL 推荐

```yaml
url-test / fallback / load-balance:
  url: http://www.gstatic.com/generate_204
  # 或
  url: http://cp.cloudflare.com/generate_204
  # 或
  url: https://connect.rom.miui.com/generate_204
```

> 提示：优先选用返回 204、体积小、延迟低的地址，避免把探测请求打到自身业务域名。

### 参数说明

| 参数 | 含义 | 推荐值 |
|------|------|--------|
| interval | 探测间隔 | 120–300 秒 |
| tolerance | 延迟容忍差 | 30–50ms |
| lazy | 仅在使用时探测 | true |
| timeout | 请求超时 | 5000ms |
| max-fails | 连续失败次数 | 3 |

### 进阶健康检查

```yaml
proxy-groups:
  - name: "智能选择"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    lazy: true
    timeout: 5000
    max-fails: 3
    proxies:
      - 节点-01
      - 节点-02
```

---

## 最佳实践

### 1. 不同场景用不同策略

```yaml
proxy-groups:
  # 流媒体——故障转移，保证不中断
  - name: "Netflix"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - Netflix-01
      - Netflix-02
      - PROXY

  # 聊天——低延迟优先
  - name: "Telegram"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 20
    proxies:
      - HK-01
      - HK-02
      - JP-01

  # 下载——负载均衡
  - name: "下载"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 300
    strategy: round-robin
    proxies:
      - 大流量-01
      - 大流量-02
```

### 2. 避免频繁切换

把 `interval` 设大、`tolerance` 设高，可以减少抖动带来的反复跳变：

```yaml
proxy-groups:
  - name: "稳定选择"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300      # 每 5 分钟测一次
    tolerance: 100    # 差 100ms 才切换
    lazy: true        # 仅活跃时探测
    proxies:
      - 节点列表
```

### 3. 与规则配合使用

```yaml
rules:
  # 流媒体
  - DOMAIN-SUFFIX,netflix.com,Netflix
  - DOMAIN-SUFFIX,disneyplus.com,Disney
  - DOMAIN-SUFFIX,hulu.com,Hulu

  # 聊天——低延迟
  - DOMAIN-SUFFIX,telegram.org,Telegram
  - DOMAIN-SUFFIX,t.me,Telegram

  # 默认
  - MATCH,PROXY
```

---

## 相关资源

- [机场导航](https://nav.clashvip.net) - VPN 节点推荐
- [Clash 教程](https://clash-for-windows.net) - 客户端下载
- [社区论坛](https://bbs.clashhub.net) - 技术讨论
- [ClashVIP](https://clashvip.net) - 订阅服务

---

## 许可证

CC BY-NC-SA 4.0 - 仅限教育用途，禁止商用
