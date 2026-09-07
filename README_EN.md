# Clash Fallback Configuration Guide

[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)
[![Stars](https://img.shields.io/github/stars/CG-spring/clash-fallback-config?style=flat-square)](https://github.com/CG-spring/clash-fallback-config/stargazers)

> Complete guide for Clash proxy group configuration - failover, load balancing, and high availability

**English** | [Chinese](README_ZH.md)

---

## Table of Contents

- [What is Failover](#what-is-failover)
- [Proxy Group Types](#proxy-group-types)
- [Failover Configuration](#failover-configuration)
- [Load Balancing](#load-balancing)
- [Health Check](#health-check)
- [Best Practices](#best-practices)

---

## What is Failover

Failover means when the primary node becomes unavailable, the system automatically switches to a backup node. In Clash, this is achieved through `proxy-groups`:

```
Primary Node (low latency)
    FAILURE DETECTED
Backup 1 -> Backup 2 -> Backup 3
    ALL FAILED
DIRECT / REJECT
```

**Benefits:**
- Improved connection stability
- No manual switching
- Automatic optimal node selection

---

## Proxy Group Types

### 1. select - Manual Selection

```yaml
proxy-groups:
  - name: "Manual"
    type: select
    proxies:
      - HongKong-01
      - HongKong-02
      - Japan-01
      - Auto-Select
      - DIRECT
```

### 2. url-test - Auto Speed Test

```yaml
proxy-groups:
  - name: "Auto-Select"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300    # Test every 300 seconds
    tolerance: 50    # Switch if latency diff > 50ms
    proxies:
      - HongKong-01
      - HongKong-02
      - Japan-01
```

### 3. fallback - Failover

```yaml
proxy-groups:
  - name: "Failover"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 300
    proxies:
      - HongKong-01     # Priority: use first
      - HongKong-02     # Switch to if 1 fails
      - Japan-01        # Switch to if 2 fails
      - DIRECT          # Direct when all fail
```

### 4. load-balance - Load Balancing

```yaml
proxy-groups:
  - name: "Load-Balance"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 300
    strategy: round-robin   # or consistent-hashing
    proxies:
      - Node-01
      - Node-02
      - Node-03
```

---

## Failover Configuration

### Basic Configuration Template

```yaml
# Clash configuration file
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
  # Main group - Failover
  - name: "PROXY"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    lazy: true
    proxies:
      - HK-Optimal
      - JP-Optimal
      - BACKUP
      - DIRECT

  # HK group - url-test
  - name: "HK-Optimal"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    proxies:
      - HK-01
      - HK-02

  # JP group - url-test
  - name: "JP-Optimal"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    proxies:
      - JP-01

  # Backup group - fallback
  - name: "BACKUP"
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

### Multi-Layer Failover

```yaml
proxy-groups:
  # Layer 1: By region
  - name: "HK-Selected"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - HK-01
      - HK-02
      - HK-03

  - name: "JP-Selected"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - JP-01
      - JP-02

  # Layer 2: Regional failover
  - name: "Asia"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - HK-Selected
      - JP-Selected

  # Layer 3: Global failover
  - name: "PROXY"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - Asia
      - US
      - EU
      - DIRECT
```

---

## Load Balancing

### Round-Robin Mode

```yaml
proxy-groups:
  - name: "RoundRobin"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 180
    strategy: round-robin
    proxies:
      - Node-01
      - Node-02
      - Node-03
```

**Use case:** High traffic scenarios, distributes load evenly.

### Consistent-Hashing Mode

```yaml
proxy-groups:
  - name: "ConsistentHash"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 180
    strategy: consistent-hashing
    proxies:
      - Node-01
      - Node-02
      - Node-03
```

**Use case:** Session persistence, same target always uses same node.

---

## Health Check

### Test URL Recommendations

```yaml
url-test / fallback / load-balance:
  url: http://www.gstatic.com/generate_204
  # or
  url: http://cp.cloudflare.com/generate_204
  # or
  url: https://connect.rom.miui.com/generate_204
```

### Parameters Explained

| Parameter | Description | Recommended |
|-----------|-------------|-------------|
| interval | Test interval | 120-300 seconds |
| tolerance | Latency difference | 30-50ms |
| lazy | Only test when active | true |
| timeout | Request timeout | 5000ms |
| max-fails | Consecutive failures | 3 |

### Advanced Health Check

```yaml
proxy-groups:
  - name: "Smart-Select"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    lazy: true
    timeout: 5000
    max-fails: 3
    proxies:
      - Node-01
      - Node-02
```

---

## Best Practices

### 1. Different Strategies for Different Use Cases

```yaml
proxy-groups:
  # Streaming - Failover
  - name: "Netflix"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - Netflix-01
      - Netflix-02
      - PROXY

  # Chat - Low Latency
  - name: "Telegram"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 20
    proxies:
      - HK-01
      - HK-02
      - JP-01

  # Downloads - Load Balance
  - name: "Download"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 300
    strategy: round-robin
    proxies:
      - Bulk-01
      - Bulk-02
```

### 2. Avoid Frequent Switching

```yaml
proxy-groups:
  - name: "Stable-Select"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300      # Test every 5 minutes
    tolerance: 100    # 100ms difference before switching
    lazy: true        # Only test when requests active
    proxies:
      - Node-List
```

### 3. Use with Rules

```yaml
rules:
  # Streaming services
  - DOMAIN-SUFFIX,netflix.com,Netflix
  - DOMAIN-SUFFIX,disneyplus.com,Disney
  - DOMAIN-SUFFIX,hulu.com,Hulu

  # Chat apps - low latency
  - DOMAIN-SUFFIX,telegram.org,Telegram
  - DOMAIN-SUFFIX,t.me,Telegram

  # Default
  - MATCH,PROXY
```

---

## Related Resources

- [Airport Navigation](https://nav.clashvip.net) - VPN recommendations
- [Clash Tutorial](https://clash-for-windows.net) - Client download
- [Community](https://bbs.clashhub.net) - Technical discussions
- [ClashVIP](https://clashvip.net) - Subscription service

---

## License

CC BY-NC-SA 4.0 - Educational use only, no commercial use
