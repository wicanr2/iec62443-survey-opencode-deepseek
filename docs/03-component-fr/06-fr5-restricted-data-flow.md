# FR 5 (RDF) — 限制資料流

FR5 回答「誰能跟誰通」——限制資料流向，確保信任邊界不被跨越。它是 Zone & Conduit 模型在組件層與系統層的技術落地：組件上做服務/port 最小化，系統上做分區與邊界防護。

下一篇：[→ FR 6 (TRE)：事件回應與稽核](07-fr6-timely-response.md)

## 1. 根本問題

認證對了 (FR1)、授權對了 (FR2)、程式沒被改 (FR3)、資料加密了 (FR4)——但整張網路是平的。一台 HMI 被入侵後，因為在同一個 VLAN，可以直接往 PLC 發 Modbus write。加密沒用，因為攻擊者是「合法連線的 HMI」。

FR5 的核心命題：誰可以跟誰講話，用什麼協定，什麼方向。 這是 defense-in-depth 的網路層基石。

| 策略 | 說明 |
|---|---|
| **Default deny** | 出廠時所有服務/port 關閉，只開放被明示允許的。這是 FR5 最重要的鐵律 |
| **最小服務** | 只執行必要的服務。不要「順便跑個 SSH server 以後 debug 用」 |
| **最小介面** | 服務不綁 `0.0.0.0`，只綁特定的內部介面 IP |
| **方向限制** | 區分 incoming / outgoing / both-way 通訊需求 |
| Segmentation | 支援 VLAN / 獨立 NIC 把流量隔離到不同 zone |

## 2. SL-C 1-4 的要求差異

> 本 FR 對應 IEC 62443-4-2 中的 **4 條 CR**（CR 5.1 – CR 5.4）。每條 CR 對特定 SL-C 等級有要求，下表為各等級的綜合摘要。CR 條號與詳細要求請參閱標準原文或 ISASecure CSA-311。
>
> [¹]: 同上。其中 CR 5.1/5.2/5.3 僅適用於 Network Device

| SL | 組件層 | 系統層 / Conduit |
|---|---|---|
| **SL-C 1** | 基本 port/服務設定 | — |
| **SL-C 2** | Default deny、ACL based on IP/port | 防火牆分區 |
| **SL-C 3** | 內建 firewall、VLAN 支援 | 工業協定 DPI（深層封包檢測） |
| **SL-C 4** | 硬體隔離 NIC | 實體隔離 + Data diode |

## 3. 軟體實作指南

### 3.1 Default Deny 的實作

```yaml
# 好的做法：出廠預設
services:
 http_api:
 enabled: false # 預設關閉
 modbus_tcp:
 enabled: false # 預設關閉
 ssh:
 enabled: false # 預設關閉

# SL-C 2 的最低要求：預設全部關閉，由客戶/整合商按需開啟
```

### 3.2 服務綁定：不用 0.0.0.0

```
反模式： app.listen(8080, "0.0.0.0") # 全介面曝露
正確做法： app.listen(8080, "10.0.2.5") # 只綁內部管理網段
或： app.listen(8080, "127.0.0.1") # 如果只有 local process 需要
```

### 3.3 通訊矩陣文件化

P8 (SG) hardening guide 必須文件化通訊需求：

| 來源 | 目的 | 協定/Port | 方向 | 用途 |
|---|---|---|---|---|
| VMS server | AMR 控制器 | Modbus TCP/502 | ↔ | 任務派送+狀態回報 |
| MES server | VMS server | HTTP/8080 | → | 工單下發 |
| AMR 控制器 | — | MQTT/8883 | ←→ | 狀態訂閱 (TLS) |

> 整合商拿到這份矩陣才知道 Conduit 上要開什麼規則。

### 3.4 Docker / 容器化部署的 RDF 實踐

容器化不等於隔離——這是 FR5 在部署層最常見的誤解。

Docker 的 `ports:`（`docker run -p`）預設綁 `0.0.0.0`——host 的**所有網路介面**都聽得到，含對外網卡。很多人以為服務跑在容器裡就自動安全了，實際上一行 `ports:` 就把它攤到全網。

**反模式**（常見的 docker-compose）：

```yaml
services:
  nginx:    ports: ["80:80"]
  backend:  ports: ["8081:8081"]   # ← 不該對外
  mongo:    ports: ["27017:27017"] # ← 不該對外
  redis:    ports: ["6379:6379"]   # ← 不該對外
```

四條 `ports:`，四條對外通道。任何一個服務有漏洞（mongo 預設無密碼、redis 無認證），攻擊者直接從外部打進來。

**正確做法**：只讓 reverse proxy 對外，內部服務全部走 Docker 內網。

```yaml
services:
  nginx:
    ports: ["80:80", "443:443"]         # 唯一對外
  backend:
    expose: ["8081"]                     # 只給同 network 的容器用
  mongo:
    expose: ["27017"]                    # 不綁 host port
  redis:
    expose: ["6379"]
```

三條原則：

1. **只有 reverse proxy（nginx/traefik）掛 `ports:`**——它是唯一被允許對外的窗口。內部服務全部用 `expose:`，只讓同一個 Docker network 內的容器看到。
2. **內部服務 bind 127.0.0.1 或不 bind host 介面**——`expose:` 不映射到 host port，容器之間靠 Docker DNS（服務名解析）溝通，根本不需要 host port。
3. **`/var/run/docker.sock` 絕不掛進容器**——這等於把 host 的 root 交出去。掛進容器的 docker.sock 讓容器可以起特權容器、掛 host 根目錄，容器的隔離邊界直接失效。

這三條對應到 IEC 62443 的話：

| Docker 面向 | Zone & Conduit 對應 |
|---|---|
| Docker 內網 (bridge network) | 一個 Zone——內部容器互信 |
| `ports:` 的 host binding | Conduit——從外部 Zone 進入的通道 |
| nginx reverse proxy | Conduit 上的安全控制點（可以在這裡做 TLS termination、IP whitelist、rate limiting） |
| 內部服務只 `expose:` | Zone 內通訊——不暴露到 Conduit 外 |

> 實務上，`ports:` 可以加 bind IP 限制：`"127.0.0.1:8081:8081"` 或 `"10.0.2.5:8081:8081"`。這比 `"8081:8081"`（等於 `0.0.0.0:8081`）安全得多——但還是比 `expose:` 多了一層 host 暴露。如果可以只用 `expose:`，就不要用 `ports:`。

## 4. 硬體支援需求

| 硬體功能 | 說明 |
|---|---|
| 獨立 NIC / VLAN offload | 把管理流量和 control 流量分在不同實體 port 或 VLAN |
| 硬體 ACL (TCAM) | 在 switch ASIC 層做 wire-speed ACL，不耗 CPU |
| Data diode | 光電物理單向：光發射器→光接收器，無反向路徑。SL-C 4 的極端場景 |

## 5. 常見不合格項目

1. 服務綁 0.0.0.0 — 全網路介面開放
2. pprof / debug port 外露 — 效能 profiling port 不該對外網開放
3. CORS `*` — 允許任何 origin 存取 API
4. 無通訊矩陣文件 — 客戶不知道該開什麼、關什麼

防禦建好了（FR1-5）。但出事了你會知道嗎？

---



## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| ACL | Access Control List | 存取控制清單，定義誰能存取什麼資源 |
| AMR | Autonomous Mobile Robot | 自主移動機器人/搬運車 |
| **CR** | Component Requirement | 組件安全需求，IEC 62443-4-2 定義 |
| CSA | Component Security Assurance | ISASecure 組件安全認證 |
| DC | Data Confidentiality | 資料機密性 (FR4) |
| DPI | Deep Packet Inspection | 深層封包檢測，辨識應用層協定內容 |
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| HMI | Human-Machine Interface | 人機介面 |
| ISASecure | ISA Security Compliance Institute | ISA 資安合規協會，營運 IEC 62443 認證方案 |
| MES | Manufacturing Execution System | 製造執行系統，管理工單與生產排程 |
| NIC | Network Interface Card | 網路介面卡 |
| PLC | Programmable Logic Controller | 可程式邏輯控制器 |
| **RDF** | Restricted Data Flow | 限制資料流 (FR5) |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| TLS | Transport Layer Security | 傳輸層安全協定，加密通訊 |
| **TRE** | Timely Response to Events | 事件及時回應 (FR6) |
| VLAN | Virtual LAN | 虛擬區域網路，邏輯隔離 |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
