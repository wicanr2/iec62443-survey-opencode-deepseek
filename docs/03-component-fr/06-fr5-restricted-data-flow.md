# FR 5 (RDF) — 限制資料流

> 一句話定位：FR5 回答「誰能跟誰通」——限制資料流向，確保信任邊界不被跨越。它是 Zone & Conduit 模型在組件層與系統層的技術落地：組件上做服務/port 最小化，系統上做分區與邊界防護。
>
> 前置：[FR 4 (DC)：資料機密性](05-fr4-data-confidentiality.md)
> 下一篇：[FR 6 (TRE)：事件回應與稽核](07-fr6-timely-response.md)

## 1. 根本問題

認證對了 (FR1)、授權對了 (FR2)、程式沒被改 (FR3)、資料加密了 (FR4)——但整張網路是平的。一台 HMI 被入侵後，因為在同一個 VLAN，可以直接往 PLC 發 Modbus write。加密沒用，因為攻擊者是「合法連線的 HMI」。

FR5 的核心命題：**誰可以跟誰講話，用什麼協定，什麼方向。** 這是 defense-in-depth 的網路層基石。

| 策略 | 說明 |
|---|---|
| **Default deny** | 出廠時所有服務/port 關閉，只開放被明示允許的。這是 FR5 最重要的鐵律 |
| **最小服務** | 只執行必要的服務。不要「順便跑個 SSH server 以後 debug 用」 |
| **最小介面** | 服務不綁 `0.0.0.0`，只綁特定的內部介面 IP |
| **方向限制** | 區分 incoming / outgoing / both-way 通訊需求 |
| **Segmentation** | 支援 VLAN / 獨立 NIC 把流量隔離到不同 zone |

## 2. SL-C 1-4 的要求差異

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
    enabled: false    # 預設關閉
  modbus_tcp:
    enabled: false    # 預設關閉
  ssh:
    enabled: false    # 預設關閉

# SL-C 2 的最低要求：預設全部關閉，由客戶/整合商按需開啟
```

### 3.2 服務綁定：不用 0.0.0.0

```
反模式：  app.listen(8080, "0.0.0.0")     # 全介面曝露
正確做法： app.listen(8080, "10.0.2.5")    # 只綁內部管理網段
或：       app.listen(8080, "127.0.0.1")   # 如果只有 local process 需要
```

### 3.3 通訊矩陣文件化

P8 (SG) hardening guide 必須文件化通訊需求：

| 來源 | 目的 | 協定/Port | 方向 | 用途 |
|---|---|---|---|---|
| VMS server | AMR 控制器 | Modbus TCP/502 | ↔ | 任務派送+狀態回報 |
| MES server | VMS server | HTTP/8080 | → | 工單下發 |
| AMR 控制器 | — | MQTT/8883 | ←→ | 狀態訂閱 (TLS) |

> 整合商拿到這份矩陣才知道 Conduit 上要開什麼規則。

## 4. 硬體支援需求

| 硬體功能 | 說明 |
|---|---|
| **獨立 NIC / VLAN offload** | 把管理流量和 control 流量分在不同實體 port 或 VLAN |
| **硬體 ACL (TCAM)** | 在 switch ASIC 層做 wire-speed ACL，不耗 CPU |
| **Data diode** | 光電物理單向：光發射器→光接收器，無反向路徑。SL-C 4 的極端場景 |

## 5. 常見不合格項目

1. **服務綁 0.0.0.0** — 全網路介面開放
2. **pprof / debug port 外露** — 效能 profiling port 不該對外網開放
3. **CORS `*`** — 允許任何 origin 存取 API
4. **無通訊矩陣文件** — 客戶不知道該開什麼、關什麼

## 6. 下一篇

> [FR 6 (TRE)：事件回應與稽核](07-fr6-timely-response.md)

防禦建好了（FR1-5）。但**出事了你會知道嗎？**

---

相關：[CONTEXT.md](../../CONTEXT.md)、[IEC 62443-4-2 官方頁](https://webstore.iec.ch/en/publication/34421)
