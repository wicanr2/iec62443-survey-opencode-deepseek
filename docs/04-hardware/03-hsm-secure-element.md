# HSM / TPM / Secure Element 選用指南

HSM、TPM、Secure Element 三者都解決同一個根本問題——「金鑰不能跟應用程式放在同一個地方」。但他們的架構、介面、成本、認證等級完全不同。本篇給出選用決策樹與實務整合指南。

> 下一篇：[ISASecure 認證體系](../05-compliance/01-isasecure-certification.md)（撰寫中）

## 1. 根本問題

金鑰存在一般 flash 或檔案系統裡，攻擊者只要拿到程式碼或記憶體傾印就能取走：
- **軟體漏洞** → read arbitrary memory → 撈到金鑰
- **物理接觸** → JTAG dump flash → 撈到金鑰
- **冷開機攻擊** → 冷凍 RAM → RAM 殘留資料 → 撈到金鑰

三者共同的答案：金鑰存在一個獨立的、專用的安全晶片裡，永不出晶片。 加解密操作在安全晶片內部完成，應用程式只拿到操作結果，拿不到金鑰本身。

## 2. 三者架構差異

| | TPM | Secure Element (SE) | External HSM |
|---|---|---|---|
| **定位** | 平台信任根（證明「這台機器是真的」） | 裝置身分與金鑰儲存 | 高安全金鑰管理與密碼運算 |
| 典型 form factor | 獨立晶片 (SPI/LPC/I2C) 或 firmware TPM | 小型 IC (I2C/SPI/SWI, < $1) | PCIe card、USB token、network appliance |
| **金鑰儲存** | ✓ (limited slots) | ✓ (數 KB EEPROM) | ✓ (大量金鑰，有專用 memory) |
| 金鑰操作在內部 | ✓ | ✓ | ✓ |
| 平台度量 (PCR) | ✓ (核心功能) | ✗ | ✗ |
| 遠端認證 (Attestation) | ✓ | 部分支援 | ✓ |
| FIPS 140-3 認證 | Level 2 常見 | Level 2 常見 | Level 3 可達 |
| **單價** | $1-3 | $0.3-1 | $100-5000+ |
| **適合場景** | Server/PC/HD | MCU/ED (嵌入式) | Payment/CA/PKI |

## 3. 選用決策樹

### 4.3 SE 內部的 key slot 管理

SE 內部 key slot 配置：

| Slot | 用途 | 屬性 |
|---|---|---|
| Slot 0 | Device primary private key (EC P-256) — TLS client cert / device authentication | 不可匯出、不可讀取 |
| Slot 1 | Backup key (optional) — 用於 key rotation | — |
| Slot 2-5 | 供應用層使用 (AES keys, HMAC keys, etc.) | — |
| OTP area | 寫入出廠資訊 (serial number, manufacturing date) | 一次性寫入 |

> SE 的金鑰產生在 SE 內部完成——私鑰從未離開晶片。開發者只拿到 public key。

### 4.4 用 SE 做 TLS Client Authentication

應用程式（Host MCU）:
 1. 發起 TLS handshake → 需要 client certificate
 2. 向 SE 發送 "sign(challenge)" 請求
 3. SE 內部: 用 Slot 0 private key 簽署 challenge
 4. SE 回傳: signature（私鑰從未離開 SE）
 5. 應用程式: 將 signature 填入 TLS client certificate verify message

## 5. TPM 的特殊功能

除了金鑰儲存，TPM 有兩個 SE 沒有的核心功能：

### 5.1 Platform Configuration Registers (PCR)

TPM 在開機過程中，每個階段會把下一階段的 binary hash 「延伸」進 PCR。例如：

PCR[0]: Boot ROM hash
PCR[1]: Bootloader hash
PCR[2]: OS kernel hash
...

**用途**：證明「這台機器是用原廠的 firmware 開機的」。遠端可以要求 TPM 簽署 PCR 值（remote attestation）——若 PCR 值與預期不符，代表 firmware 被改過。

### 5.2 密封儲存 (Sealing)

TPM 可以用 PCR 值當作解封條件：
- "只有當 PCR[0-3] 等於這些預期值時，才能解開這個 blob"
- 如果 firmware 被改過（PCR 值變了），金鑰 blob 就解不開

## 6. 成本 vs 安全矩陣

| 場景 | 方案 | 成本 | 達到 SL-C |
|---|---|---|---|
| MCU 無額外晶片 | 軟體實作 + flash 儲存金鑰 | $0 | SL-C 1-2 |
| MCU + SE (I2C) | ATECC608 等 | +$0.5 | SL-C 3 |
| Linux SBC + TPM | TPM 2.0 chip | +$2 | SL-C 3 |
| Linux SBC + TPM + SE | 雙晶片 | +$3 | SL-C 3-4 |
| Server + PCIe HSM | Thales/Gemalto HSM | +$500-5000 | SL-C 4 |

## 7. 常見陷阱

1. 買了 SE 但私鑰還是用軟體產生再匯入 SE — SE 的價值在「私鑰從未離開 SE」
2. I2C bus 沒保護 — I2C 是 shared bus，攻擊者可以在 I2C 上 sniff/spoof。SE 支援 encrypted I2C channel
3. 只用 TPM 的 storage key 功能、沒用 PCR — 等於把 TPM 當成貴的 flash
4. Key slot 用完沒規劃 — SE 只有少量 key slot（4-16 個），需預先規劃用途


- TPM = 平台信任（PCR + attestation），適合 HD (Host Device)
- SE = 裝置金鑰儲存，適合 ED (Embedded Device)，成本極低
- HSM = 企業級金鑰管理，適合 CA / PKI / payment
- SE + TPM 雙晶片可達到 SL-C 4 的硬體金鑰保護
- 關鍵原則：金鑰永不離開安全晶片——產生、簽署、解密全在晶片內

## 9. Phase 4 結束，下一篇

硬體安全三篇完成。下一篇進入 Phase 5：合規與認證實務 → [ISASecure 認證體系](../05-compliance/01-isasecure-certification.md)（撰寫中）

---



## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| AES | Advanced Encryption Standard | 進階加密標準，對稱式加密演算法 |
| CA | Certificate Authority | 憑證授權中心，簽發數位憑證 |
| **ED** | Embedded Device | 嵌入式裝置組件 (IEC 62443-4-2 組件類型) |
| **HD** | Host Device | 主機裝置組件 (IEC 62443-4-2 組件類型) |
| HMAC | Hash-based Message Authentication Code | 雜湊訊息鑑別碼，驗證完整性+來源 |
| HSM | Hardware Security Module | 硬體安全模組，專用加密金鑰管理硬體 |
| I2C | Inter-Integrated Circuit | 晶片間序列通訊匯流排 |
| ISASecure | ISA Security Compliance Institute | ISA 資安合規協會，營運 IEC 62443 認證方案 |
| JTAG | Joint Test Action Group | 聯合測試行動組，晶片除錯介面標準 |
| MCU | Microcontroller Unit | 微控制器，嵌入式系統的核心晶片 |
| OS | Operating System | 作業系統 |
| OTP | One-Time Programmable | 一次性可程式化記憶體 (eFuse 類) |
| PCR | Platform Configuration Register | 平台組態暫存器，TPM 的開機度量記錄 |
| PKI | Public Key Infrastructure | 公開金鑰基礎設施，管理憑證發行/撤銷/輪替 |
| SE | Secure Element | 安全元件，晶片級金鑰儲存 (如 ATECC608) |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| SPI | Serial Peripheral Interface | 序列週邊介面，晶片間高速通訊 |
| TLS | Transport Layer Security | 傳輸層安全協定，加密通訊 |
| TPM | Trusted Platform Module | 可信平台模組，平台身分與金鑰儲存 |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)

---

## 版本資訊

- **基於標準**：IEC 62443-4-2:2019 (ED1)、IEC 62443-4-1:2018
- **認證方案**：ISASecure CSA 1.0.0
- **知識庫版本**：v0.1.0（2026-06-30）

> 詳細演進見 [CHANGELOG.md](../../CHANGELOG.md)

