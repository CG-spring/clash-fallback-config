# Clash Fallback Configuration Guide

[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)

> Complete guide for Clash proxy group failover and load balancing

## Proxy Group Types

### url-test - Auto Speed Test

```yaml
proxy-groups:
  - name: "Auto"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300
    proxies:
      - Node1
      - Node2
```

### fallback - Failover

```yaml
proxy-groups:
  - name: "Failover"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - Node1
      - Node2
```

## Resources

- https://nav.clashvip.net
- https://clash-for-windows.net

## License

CC BY-NC-SA 4.0
