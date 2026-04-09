# Clash 鏁呴殰杞Щ閰嶇疆瀹屽叏鎸囧崡

[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/CG-spring/clash-fallback-config.svg?style=flat-square)](https://github.com/CG-spring/clash-fallback-config/stargazers)

> Clash Proxy Group 楂樺彲鐢ㄩ厤缃?- 鑷姩鏁呴殰杞Щ銆佽礋杞藉潎琛°€佸欢杩熸祴璇?> 
> 鎸佺画鏇存柊涓?| 鏈€鍚庢洿鏂? 2026-04-09

**涓枃** | **[English](README_EN.md)**

---

## 鐩綍

- [浠€涔堟槸鏁呴殰杞Щ](#浠€涔堟槸鏁呴殰杞Щ)
- [Proxy Group 绫诲瀷](#proxy-group-绫诲瀷)
- [鏁呴殰杞Щ閰嶇疆](#鏁呴殰杞Щ閰嶇疆)
- [璐熻浇鍧囪　閰嶇疆](#璐熻浇鍧囪　閰嶇疆)
- [鍋ュ悍妫€鏌ラ厤缃甝(#鍋ュ悍妫€鏌ラ厤缃?
- [鏈€浣冲疄璺礭(#鏈€浣冲疄璺?

---

## 浠€涔堟槸鏁呴殰杞Щ

鏁呴殰杞Щ锛團ailover锛夋槸鎸囧綋涓昏妭鐐逛笉鍙敤鏃讹紝鑷姩鍒囨崲鍒板鐢ㄨ妭鐐圭殑鏈哄埗銆傚湪 Clash 涓紝閫氳繃 `proxy-groups` 瀹炵幇锛?
```
涓昏妭鐐?(浣庡欢杩? 
    鈫?妫€娴嬪け璐?澶囩敤鑺傜偣 1 鈫?澶囩敤鑺傜偣 2 鈫?澶囩敤鑺傜偣 3
    鈫?鍏ㄩ儴澶辫触
鐩磋繛/鎷掔粷
```

**浼樺娍锛?*
- 鎻愰珮杩炴帴绋冲畾鎬?- 鍑忓皯鎵嬪姩鍒囨崲
- 鑷姩閫夋嫨鏈€浼樿妭鐐?
---

## Proxy Group 绫诲瀷

### 1. select - 鎵嬪姩閫夋嫨

```yaml
proxy-groups:
  - name: "鎵嬪姩閫夋嫨"
    type: select
    proxies:
      - 棣欐腐鑺傜偣1
      - 棣欐腐鑺傜偣2
      - 鏃ユ湰鑺傜偣
      - 鑷姩閫夋嫨
      - DIRECT
```

### 2. url-test - 鑷姩娴嬮€熼€夋嫨

```yaml
proxy-groups:
  - name: "鑷姩閫夋嫨"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300  # 姣?300 绉掓祴閫熶竴娆?    tolerance: 50  # 寤惰繜宸?50ms 鍐呬笉鍒囨崲
    proxies:
      - 棣欐腐鑺傜偣1
      - 棣欐腐鑺傜偣2
      - 鏃ユ湰鑺傜偣
```

### 3. fallback - 鏁呴殰杞Щ

```yaml
proxy-groups:
  - name: "鏁呴殰杞Щ"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 300
    proxies:
      - 棣欐腐鑺傜偣1  # 浼樺厛浣跨敤
      - 棣欐腐鑺傜偣2  # 鑺傜偣1澶辫触鏃跺垏鎹?      - 鏃ユ湰鑺傜偣   # 鑺傜偣2澶辫触鏃跺垏鎹?      - DIRECT     # 鍏ㄩ儴澶辫触鐩磋繛
```

### 4. load-balance - 璐熻浇鍧囪　

```yaml
proxy-groups:
  - name: "璐熻浇鍧囪　"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 300
    strategy: round-robin  # 鎴?consistent-hashing
    proxies:
      - 棣欐腐鑺傜偣1
      - 棣欐腐鑺傜偣2
      - 鏃ユ湰鑺傜偣
```

---

## 鏁呴殰杞Щ閰嶇疆

### 鍩虹閰嶇疆妯℃澘

```yaml
# Clash 閰嶇疆鏂囦欢
proxies:
  - name: "棣欐腐01"
    type: ss
    server: hk1.example.com
    port: 443
    cipher: aes-256-gcm
    password: "password"
    
  - name: "棣欐腐02"
    type: ss
    server: hk2.example.com
    port: 443
    cipher: aes-256-gcm
    password: "password"
    
  - name: "鏃ユ湰01"
    type: ss
    server: jp1.example.com
    port: 443
    cipher: aes-256-gcm
    password: "password"

proxy-groups:
  # 涓荤粍 - 鏁呴殰杞Щ
  - name: "PROXY"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    lazy: true  # 鏃犺姹傛椂涓嶆娴?    proxies:
      - 棣欐腐鏈€浼?      - 鏃ユ湰鏈€浼?      - 澶囩敤缁?      - DIRECT

  # 棣欐腐缁?- 鑷姩閫夐€?  - name: "棣欐腐鏈€浼?
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    proxies:
      - 棣欐腐01
      - 棣欐腐02

  # 鏃ユ湰缁?- 鑷姩閫夐€?  - name: "鏃ユ湰鏈€浼?
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    proxies:
      - 鏃ユ湰01

  # 澶囩敤缁?- 鏁呴殰杞Щ
  - name: "澶囩敤缁?
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 300
    proxies:
      - 鏃ユ湰01
      - DIRECT

rules:
  - GEOIP,CN,DIRECT
  - MATCH,PROXY
```

### 杩涢樁閰嶇疆 - 澶氬眰鏁呴殰杞Щ

```yaml
proxy-groups:
  # 绗竴灞傦細鎸夊湴鍖哄垎缁?  - name: "棣欐腐绮鹃€?
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - 棣欐腐01
      - 棣欐腐02
      - 棣欐腐03

  - name: "鏃ユ湰绮鹃€?
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - 鏃ユ湰01
      - 鏃ユ湰02

  - name: "鏂板姞鍧＄簿閫?
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 30
    proxies:
      - 鏂板姞鍧?1
      - 鏂板姞鍧?2

  # 绗簩灞傦細鍦板尯鏁呴殰杞Щ
  - name: "浜氭床缁?
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - 棣欐腐绮鹃€?      - 鏃ユ湰绮鹃€?      - 鏂板姞鍧＄簿閫?
  # 绗笁灞傦細鍏ㄧ悆鏁呴殰杞Щ
  - name: "PROXY"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - 浜氭床缁?      - 缇庡浗缁?      - 娆ф床缁?      - DIRECT
```

---

## 璐熻浇鍧囪　閰嶇疆

### Round-Robin 妯″紡

```yaml
proxy-groups:
  - name: "璐熻浇鍧囪　"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 180
    strategy: round-robin  # 杞妯″紡
    proxies:
      - 鑺傜偣1
      - 鑺傜偣2
      - 鑺傜偣3
```

**鐗圭偣锛?* 璇锋眰渚濇鍒嗛厤鍒颁笉鍚岃妭鐐癸紝閫傚悎澶ф祦閲忓満鏅€?
### Consistent-Hashing 妯″紡

```yaml
proxy-groups:
  - name: "涓€鑷存€у搱甯?
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 180
    strategy: consistent-hashing
    proxies:
      - 鑺傜偣1
      - 鑺傜偣2
      - 鑺傜偣3
```

**鐗圭偣锛?* 鍚屼竴鐩爣濮嬬粓浣跨敤鍚屼竴鑺傜偣锛岄€傚悎闇€瑕佷細璇濅繚鎸佺殑鍦烘櫙銆?
---

## 鍋ュ悍妫€鏌ラ厤缃?
### 妫€娴?URL 閫夋嫨

```yaml
# 鎺ㄨ崘鐨勬娴?URL
url-test / fallback / load-balance:
  url: http://www.gstatic.com/generate_204
  # 鎴?  url: http://cp.cloudflare.com/generate_204
  # 鎴?  url: https://connect.rom.miui.com/generate_204
```

### 妫€娴嬪弬鏁拌鏄?
| 鍙傛暟 | 璇存槑 | 鎺ㄨ崘鍊?|
|------|------|--------|
| interval | 妫€娴嬮棿闅?| 120-300绉?|
| tolerance | 寤惰繜瀹瑰樊 | 30-50ms |
| lazy | 鎳掑姞杞?| true |
| timeout | 瓒呮椂鏃堕棿 | 5000ms |

### 瀹屾暣鍋ュ悍妫€鏌ラ厤缃?
```yaml
proxy-groups:
  - name: "鏅鸿兘閫夋嫨"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 180
    tolerance: 50
    lazy: true
    timeout: 5000
    max-fails: 3  # 杩炵画3娆″け璐ユ爣璁颁负涓嶅彲鐢?    proxies:
      - 鑺傜偣1
      - 鑺傜偣2
```

---

## 鏈€浣冲疄璺?
### 1. 鍒嗗満鏅娇鐢ㄤ笉鍚岀瓥鐣?
```yaml
proxy-groups:
  # 娴佸獟浣?- 鏁呴殰杞Щ
  - name: "Netflix"
    type: fallback
    url: http://www.gstatic.com/generate_204
    interval: 180
    proxies:
      - Netflix涓撶敤鑺傜偣1
      - Netflix涓撶敤鑺傜偣2
      - PROXY

  # 鑱婂ぉ杞欢 - 浣庡欢杩熶紭鍏?  - name: "Telegram"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 120
    tolerance: 20
    proxies:
      - 棣欐腐01
      - 棣欐腐02
      - 鏃ユ湰01

  # 涓嬭浇 - 璐熻浇鍧囪　
  - name: "涓嬭浇"
    type: load-balance
    url: http://www.gstatic.com/generate_204
    interval: 300
    strategy: round-robin
    proxies:
      - 澶ф祦閲忚妭鐐?
      - 澶ф祦閲忚妭鐐?
```

### 2. 閬垮厤棰戠箒鍒囨崲

```yaml
proxy-groups:
  - name: "绋冲畾閫夋嫨"
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300      # 5鍒嗛挓妫€娴嬩竴娆?    tolerance: 100     # 100ms 鍐呬笉鍒囨崲
    lazy: true         # 鏃犺姹備笉妫€娴?    proxies:
      - 鑺傜偣鍒楄〃
```

### 3. 閰嶅悎瑙勫垯浣跨敤

```yaml
rules:
  # 娴佸獟浣撹蛋涓撶敤缁?  - DOMAIN-SUFFIX,netflix.com,Netflix
  - DOMAIN-SUFFIX,disneyplus.com,Disney
  
  # 鑱婂ぉ杞欢璧颁綆寤惰繜缁?  - DOMAIN-SUFFIX,telegram.org,Telegram
  - DOMAIN-SUFFIX,t.me,Telegram
  
  # 鍏朵粬璧颁富缁?  - MATCH,PROXY
```

---

## 鎺ㄨ崘璧勬簮

- [鏈哄満瀵艰埅](https://nav.clashvip.net) - 浼樿川鏈哄満鎺ㄨ崘
- [Clash 鏁欑▼](https://clash-for-windows.net) - 瀹㈡埛绔笅杞?- [鐢ㄦ埛绀惧尯](https://bbs.clashhub.net) - 鎶€鏈氦娴?- [ClashVIP](https://clashvip.net) - 瑙勫垯璁㈤槄

---

## License

CC BY-NC-SA 4.0 - 浠呬緵瀛︿範浜ゆ祦锛岀姝㈠晢鐢?