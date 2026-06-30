# 物理防篡改與除錯埠管理

軟體安全做好了（FR1-7），但攻擊者如果物理接觸你的設備——打開機殼、接上 JTAG、拉出 I2C bus——軟體防禦全部失效。本篇回答：如何偵測物理入侵、如何回應、如何管理除錯介面的生命週期。

下一篇：[→ HSM / TPM / Secure Element 選用指南](03-hsm-secure-element.md)

## 1. 根本問題

你的 PLC 有 Secure Boot、有 TLS、有 RBAC。攻擊者直接走進產線：「我是維修人員。」打開機櫃，拿出筆電，插上 JTAG debugger 接上你的 MCU 的 SWD 腳位——所有軟體防禦瞬間歸零。

物理攻擊不是科幻。工控設備通常放在工廠現場，不像 server farm 有保全門禁。Stuxnet 就是透過 USB 跳過 air-gap 的實例——攻擊者可能不需要物理接觸設備本身，但內部人員/維修承包商可以。

IEC 62443-4-2 的 FR3 (SI) 對物理篡改有明確要求（SL-C 3+）。

## 2. 防篡改的兩層：偵測 + 回應

### 2.1 篡改偵測 (Tamper Detection)

| 方法 | 說明 | 成本 |
|---|---|---|
| **機殼開啟開關** | 機械微動開關 or 霍爾感測器：機殼打開→觸發中斷 | 極低 |
| 導電網格 (Tamper Mesh) | 在 PCB 或機殼內層舖導電線路網格，若被鑽孔或切割→斷路觸發。FIPS 140-3 Level 3+ 要求 | 中 |
| **環境感測器** | 溫度、濕度、加速度異常（如被搬運、被加熱除膠）→ 觸發 | 低 |
| **光感測器** | 機殼打開→環境光進入→觸發（簡單有效的 PTI detection） | 極低 |
| 電壓/頻率監測 (Glitch detector) | 監測供電電壓與時脈，若出現異常脈衝（voltage glitching attack）→ 觸發 | 晶片級 |

### 2.2 篡改回應 (Tamper Response)

偵測到之後做什麼：

| 回應 | 說明 | 適用 SL |
|---|---|---|
| **記錄事件** | 把 tamper event 寫入 secure log（含 timestamp） | SL-C 2+ |
| **抹除金鑰** | 立即 zeroize SE/HSM 內的所有私鑰—攻擊者就算拿到硬體也拿不到金鑰 | SL-C 3+ |
| **鎖死裝置** | 裝置進入永久鎖定狀態，需送回原廠才能解鎖 | SL-C 4 |
| **觸發告警** | 向中央 SIEM 發送 tamper alert（若裝置還能通訊） | SL-C 3+ |

> 核心原則：tamper response 必須在斷電情況下也能運作。 也就是說，zeroize 金鑰的能量來源必須是電池或電容，不能完全依賴主電源（因為攻擊者可能先斷電再開機殼）。

## 3. 除錯介面生命週期管理

JTAG / SWD / UART debug console 是開發期最常用的工具，也是量產後最大的安全漏洞。

### 3.1 生命週期狀態機

``````

### 3.2 各階段的管控

| 階段 | 除錯介面狀態 | 說明 |
|---|---|---|
| **開發期** | 全開 | R&D 需要 JTAG + UART。所有開發板都有 debug header |
| **生產期** | 有限開 | 產測需要用 JTAG 燒錄 firmware，但完成後鎖定 |
| **部署/營運** | 關閉或認證 | SL-C 2：禁用 JTAG；SL-C 3+：禁用 JTAG 或只允許用 signed challenge-response 解鎖 |
| **退役** | 永久禁用 + 清除 | 防退役設備被拿來逆向工程 |

### 3.3 JTAG Lock 的實作方式

| 方法 | 安全性 | 說明 |
|---|---|---|
| Fuse-based disable | | 燒斷 JTAG enable fuse → 物理上無法恢復。缺點：量產後不能再 debug |
| Password-based lock | ☆ | 用 password unlock JTAG。缺點：password 可能洩漏或被暴力破解 |
| Challenge-response unlock | | 裝置發隨機 challenge，使用者用 private key 簽署 response。最安全 |
| Permanent disable | | 產品壽命結束，燒斷所有 debug fuses + erase all keys |

## 4. 實體安全設計原則

### 4.1 PCB 層級的防護

| 原則 | 說明 |
|---|---|
| 隱藏 test point | 不要標示 JTAG/SWD pin 的 silk screen（TP1, TP2 而非 SWCLK, SWDIO） |
| 移除 debug header | 量產 PCB 不安裝 JTAG 排針——改用 test pad 或 pogo pin 接觸 |
| Secure enclosure | 機殼用 special screw (tamper-proof Torx / pentalobe)，增加開啟難度 |
| PCB potting | 樹脂灌封：攻擊者必須破壞 PCB 才能接觸晶片（無法復原的證據） |

### 4.2 晶片層級的防護

| 功能 | 說明 |
|---|---|
| Secure flash (on-die) | 晶片內建 flash（非外部 SPI flash），防止解焊讀取 |
| Active shield (晶片金屬層) | 晶片頂層金屬網格，鑽孔→觸發 tamper |
| Voltage/temperature sensor | 偵測 fault injection attack（高/低電壓、過冷/過熱、電磁脈衝） |
| Memory encryption (on-the-fly) | 外部 RAM 的內容即時加解密，防止 probing |

## 5. 常見不合格項目

1. 量產 firmware 中 JTAG/SWD 未禁用 — 最常見的 fail
2. Debug UART 在 release build 仍輸出機密資訊（密鑰、token 等）
3. 機殼用標準十字螺絲 — 任何人拿到螺絲起子就能開
4. 金鑰存在一般 flash，無 tamper zeroize — 物理滲透後照拿
5. 服務手冊公開 debug 介面的 pinout — 幫攻擊者省時間


- 物理安全是軟體安全的基石：軟體防禦對物理接觸完全無效
- 防篡改 = 偵測（開關、網格、感測器）+ 回應（記錄、抹金鑰、鎖裝置）
- JTAG/SWD 生命週期：開發開→生產鎖→營運關→退役永久禁用
- Tamper response 能量來源必須獨立於主電源（電池/電容）
- 零成本方案：移除 JTAG 排針 + tamper-proof 螺絲 + 光感測器

物理防護做好了。下一個硬體安全的基石：金鑰要存在哪？用什麼晶片來做安全運算？

---



## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| HSM | Hardware Security Module | 硬體安全模組，專用加密金鑰管理硬體 |
| I2C | Inter-Integrated Circuit | 晶片間序列通訊匯流排 |
| JTAG | Joint Test Action Group | 聯合測試行動組，晶片除錯介面標準 |
| MCU | Microcontroller Unit | 微控制器，嵌入式系統的核心晶片 |
| PLC | Programmable Logic Controller | 可程式邏輯控制器 |
| RBAC | Role-Based Access Control | 基於角色的存取控制 |
| SE | Secure Element | 安全元件，晶片級金鑰儲存 (如 ATECC608) |
| SI | System Integrity | 系統完整性 (FR3) |
| SIEM | Security Information and Event Management | 資安資訊與事件管理系統 |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| SPI | Serial Peripheral Interface | 序列週邊介面，晶片間高速通訊 |
| SWD | Serial Wire Debug | 序列線除錯，ARM MCU 的除錯介面 |
| TLS | Transport Layer Security | 傳輸層安全協定，加密通訊 |
| TPM | Trusted Platform Module | 可信平台模組，平台身分與金鑰儲存 |
| UART | Universal Asynchronous Receiver-Transmitter | 通用非同步收發器，序列通訊 |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)

---

## 版本資訊

- **基於標準**：IEC 62443-4-2:2019 (ED1)、IEC 62443-4-1:2018
- **認證方案**：ISASecure CSA 1.0.0
- **知識庫版本**：v0.1.0（2026-06-30）

> 詳細演進見 [CHANGELOG.md](../../CHANGELOG.md)

