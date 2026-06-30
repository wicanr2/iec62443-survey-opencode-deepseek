# FR 6 (TRE) — 事件及時回應

FR6 回答「出事了你知道嗎」——安全事件的記錄、匯出、告警、不可否認性。防禦再好（FR1-5），如果沒有監控和告警，攻擊者可能在你系統裡待了數月而你渾然不知。FR6 是把「事後才知道」變成「及時發現」的關鍵。

下一篇：[→ FR 7 (RA)：資源可用性](08-fr7-resource-availability.md)

## 1. 根本問題

一個典型的工控被入侵時間線：攻擊者從企業網路釣魚成功 → 橫向移動到 DMZ → 進入控制區 → 在 PLC firmware 植入 rootkit。這整個過程可能長達數週，但因為沒有安全事件記錄與監控，IT/OT 團隊在入侵後數月才從第三方得知。

FR6 的核心命題：安全事件的記錄、匯總、告警要能夠支援及時偵測與回應。

## 2. SL-C 1-4 的要求差異

> 本 FR 對應 IEC 62443-4-2 中的 **2 條 CR**（CR 6.1 – CR 6.2）。每條 CR 對特定 SL-C 等級有要求，下表為各等級的綜合摘要。CR 條號與詳細要求請參閱標準原文或 ISASecure CSA-311。
>
> [¹]: 同上

| SL | 事件記錄 | 匯出 | 告警 | 稽核軌跡 |
|---|---|---|---|---|
| **SL-C 1** | 基本事件記錄 | — | — | — |
| **SL-C 2** | 安全相關事件記錄 | 可匯出 (如 syslog) | 本機告警 | 基本 |
| **SL-C 3** | 集中式監控 | 即時匯出 (syslog over TLS) | 中央 SIEM 整合 | 防篡改日誌 + NTP |
| **SL-C 4** | 自動化事件響應 (SOAR) | — | — | Forensic readiness |

## 3. 軟體實作指南

### 3.1 該記錄什麼？

| 事件類別 | 事件範例 | 必要欄位 |
|---|---|---|
| **認證事件** | 登入成功/失敗、登出、session timeout | 帳號、來源 IP、時間戳、結果 |
| **授權事件** | 存取被拒、權限變更、角色變更 | 帳號、請求資源、結果 |
| **系統變更** | 設定變更、firmware 更新、新增/刪除使用者 | 操作者、變更前後值 |
| **異常事件** | CPU/memory 異常飆高、非預期 reboot、安全功能失效 | 異常類型、數值 |
| **通訊事件** | Conduit rule violation、來源 IP 不在白名單 | 來源/目的 IP、協定、埠號 |

### 3.2 日誌格式

```
建議格式: syslog (RFC 5424)
優先度: 使用 severity level (0-7)

範例 (JSON over syslog):
{
 "timestamp": "2026-06-30T15:04:05Z",
 "event": "AUTH_FAIL",
 "user": "operator2",
 "src_ip": "10.0.1.55",
 "reason": "invalid_password",
 "attempts": 3,
 "device_id": "AMR-04"
}
```

### 3.3 時間同步（為什麼 NTP 是安全需求）

如果你的 HMI 時間是 15:00、PLC 時間是 14:58、VMS 時間是 15:02，三份日誌對不起來——無法重建事件時間線。NTP 同步是所有安全事件記錄的前提。

| SL | NTP 要求 |
|---|---|
| SL-C 1-2 | 可選 |
| SL-C 3+ | 強制 NTP / PTP |

### 3.4 不可否認性

「我沒發過那個指令」—不可否認性確保操作不能被事後否認：

| 機制 | 說明 |
|---|---|
| 稽核軌跡完整性 | 日誌的每一筆 entry 加上 HMAC，確保沒被事後修改 |
| 使用者作業簽章（SL-C 4） | 關鍵操作（如緊急停止）由操作者數位簽章後才執行 |

## 4. 硬體支援需求

| 硬體功能 | 說明 |
|---|---|
| RTC (Real-Time Clock) + 電池 | 斷電後時間不跑掉 |
| Secure storage for logs | 防篡改的日誌儲存區（獨立的 flash partition、或 SE/HSM 保護） |
| Hardware watchdog | 若監控 daemon crash → 自動重啟 |

## 5. 嵌入式裝置的 FR6 困境與解法

| 困境 | 解法 |
|---|---|
| **儲存空間小**（128KB flash） | 只記錄安全相關事件，不記錄 debug。用 circular buffer 只保留最後 N 筆 |
| 沒有 NTP 來源 | factory network 必須佈署至少一台 NTP server |
| **不能即時匯出**（離線環境） | 階段性匯出：維修工程師帶筆電來，透過 USB 下載日誌 |

## 6. 常見不合格項目

1. 安全事件「有記錄但沒人看」 — 記錄的行為跟沒記錄一樣
2. **無 NTP** — 日誌時間戳無法對齊、無法重建事件順序
3. 日誌無完整性保護 — 攻擊者可以清掉自己的足跡
4. 只記錯誤、不記憶功 — 安全稽核需要完整軌跡（誰做了什麼、成功與否）


- FR6 = 記錄 + 匯出 + 告警 + 不可否認性，四層從偵測到回應
- NTP 不只是 convenience，是安全需求的基礎
- 嵌入式裝置的資源限制可用 circular buffer + 階段匯出解決

記錄都有了（FR6）。但系統被 DoS 打掛了怎麼辦？

---



## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| AMR | Autonomous Mobile Robot | 自主移動機器人/搬運車 |
| **CR** | Component Requirement | 組件安全需求，IEC 62443-4-2 定義 |
| CSA | Component Security Assurance | ISASecure 組件安全認證 |
| DoS | Denial of Service | 服務阻斷攻擊，耗盡系統資源使其無法回應 |
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| HMAC | Hash-based Message Authentication Code | 雜湊訊息鑑別碼，驗證完整性+來源 |
| HMI | Human-Machine Interface | 人機介面 |
| HSM | Hardware Security Module | 硬體安全模組，專用加密金鑰管理硬體 |
| ISASecure | ISA Security Compliance Institute | ISA 資安合規協會，營運 IEC 62443 認證方案 |
| NTP | Network Time Protocol | 網路時間協定，同步設備時鐘 |
| PLC | Programmable Logic Controller | 可程式邏輯控制器 |
| **RA** | Resource Availability | 資源可用性 (FR7) |
| **RDF** | Restricted Data Flow | 限制資料流 (FR5) |
| RTC | Real-Time Clock | 即時時鐘，斷電後仍維持時間 |
| SE | Secure Element | 安全元件，晶片級金鑰儲存 (如 ATECC608) |
| SIEM | Security Information and Event Management | 資安資訊與事件管理系統 |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| SOAR | Security Orchestration Automation and Response | 安全自動化協作與回應 |
| TLS | Transport Layer Security | 傳輸層安全協定，加密通訊 |
| **TRE** | Timely Response to Events | 事件及時回應 (FR6) |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)

---

## 版本資訊

- **基於標準**：IEC 62443-4-2:2019 (ED1)、IEC 62443-4-1:2018
- **認證方案**：ISASecure CSA 1.0.0
- **知識庫版本**：v0.1.0（2026-06-30）

> 詳細演進見 [CHANGELOG.md](../../CHANGELOG.md)

