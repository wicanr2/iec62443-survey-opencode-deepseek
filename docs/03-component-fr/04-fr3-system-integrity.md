# FR 3 (SI) — 系統完整性

> 一句話定位：FR3 回答「系統有沒有被偷改」——從開機的 bootloader、運行中的 firmware、到網路傳來的輸入，每一層都要驗證內容是原廠的、沒被篡改的。對嵌入式裝置而言，這是整個 FR 集中硬體倚賴最深的一條。
>
> 前置：[FR 2 (UC)：使用控制與授權](03-fr2-use-control.md)
> 下一篇：[FR 4 (DC)：資料機密性與金鑰管理](05-fr4-data-confidentiality.md)

## 1. 根本問題

三個層級，三種威脅：

| 層級 | 保護對象 | 威脅 |
|---|---|---|
| **開機時** | Bootloader → OS/FW | 攻擊者換掉你的 firmware，塞入後門。開機後一切防禦都白做 |
| **運行中** | 執行中的 code / 參數 | runtime exploitation（buffer overflow, ROP chain）篡改記憶體或暫存器 |
| **傳輸/儲存** | 更新檔、日誌、備份 | 中間人攻擊篡改 OTA update；離線修改 flash 內容 |

> SI 的鐵律：**信任必須有起點。** 如果 bootloader 本身可能被篡改，它驗證的 firmware 簽章就沒有意義。這是硬體信任根 (Root of Trust) 存在的根本原因——詳見 [硬體信任根專題](../04-hardware/01-hardware-root-of-trust.md)。

## 2. SL-C 1-4 的要求差異

> 本 FR 對應 IEC 62443-4-2 中的 **14 條 CR**（CR 3.1 – CR 3.14）。每條 CR 對特定 SL-C 等級有要求，下表為各等級的綜合摘要。CR 條號與詳細要求請參閱標準原文或 ISASecure CSA-311。
>
> [¹]: 同上。CR 3.12 Secure Provisioning、CR 3.13 Boot Process Integrity、CR 3.14 Data-at-Rest Integrity 在部分來源中為補充項目


| SL | 開機完整性 | 韌體/軟體完整性 | 輸入驗證 | 惡意程式防護 |
|---|---|---|---|---|
| **SL-C 1** | — | CRC / checksum（只防意外） | 基本驗證 | — |
| **SL-C 2** | — | 簽章驗證（防蓄意） | 輸入型別/長度驗證 | — |
| **SL-C 3** | Secure Boot | 簽章 + anti-rollback | 白名單驗證 | Malware detection |
| **SL-C 4** | HW root of trust | 即時 runtime integrity check | 全自動輸入過濾 | Malware prevention |

## 3. 軟體實作指南

### 3.1 Secure Boot 信任鏈

 <p align="center"><img src="../../img/14-secureboot-chain.svg" width="520" alt="Secure Boot 信任鏈"></p>

每一層驗證下一層的簽章後才交給控制權。任何一層驗證失敗 → 不開機 / 進 recovery mode。

### 3.2 韌體簽章驗證

```
Build process:
  1. 編譯 firmware
  2. 對 binary 做 hash (SHA-256)
  3. 用 release private key 簽署 hash → 產生 .sig
  4. 將 binary + .sig + public key (embedded in bootloader) 打包成 update package

Device side (bootloader):
  1. 讀取 firmware binary，計算 hash
  2. 讀取 .sig，用內嵌的 public key 驗證簽章
  3. 簽章有效且 anti-rollback counter 正確 → 開機
  4. 任一步失敗 → 拒絕開機 / 進 recovery
```

### 3.3 輸入驗證：工控特有陷阱

| 輸入來源 | 驗證項目 |
|---|---|
| Modbus TCP 封包 | Function code 範圍、register address 範圍、length 不超出允許的 register block |
| HTTP API (REST) | Content-Type 檢查、payload 長度上限、JSON schema validation、SQL injection/XSS sanitize |
| OTA update payload | 簽章驗證 + 版本號 + anti-rollback check + 完整性 hash |
| CAN bus frame | CAN ID 白名單、DLC 長度驗證（CAN 允許 DLC > 8 bytes，那是攻擊用的） |
| Serial / UART debug | 量產 firmware 禁用 debug 輸入，或限於認證後的 session |

### 3.4 反回滾 (Anti-rollback)

見 [P7 SUM 更新管理](../02-sdlc/05-update-patch-management.md) 的詳細說明。

核心：版本號 monotonic counter。軟體層 counter 可被物理攻擊者改 — SL-C 3+ 需要硬體支援（eFuse / secure counter）。

## 4. 硬體支援需求

| 硬體功能 | 對應 SL-C | 說明 |
|---|---|---|
| **Boot ROM (mask ROM)** | SL-C 3-4 | 不可修改的第一段 boot code，驗證下一段的簽章 |
| **Secure Element / eFuse** | SL-C 3-4 | 存放 public key hash (root of trust anchor) |
| **MPU / MMU** | SL-C 2-4 | 記憶體保護：code region read-only、data region non-executable |
| **Tamper detection** | SL-C 4 | 見 [04-hardware/](../04-hardware/) 專題 |
| **JTAG lock** | SL-C 2-4 | 量產後禁用除錯介面或強制認證 |

## 5. 組件類型特化要點

| 類型 | SI 重點 |
|---|---|
| **SA** | 輸入驗證、防止 SQL/command injection、相依 library 弱點掃描；倚賴 Host OS 的 secure boot |
| **ED** | 核心：Secure Boot + FW 簽章驗證 + JTAG lock + anti-rollback。所有 SI 功能要自己實作 |
| **HD** | Secure Boot (UEFI Secure Boot / verified boot) + file integrity (AIDE, dm-verity) |
| **ND** | Firmware integrity + 管理介面的輸入驗證 |

## 6. 常見不合格項目

1. **更新無簽章驗證** — 最常見的 fail：firmware 更新檔放個 HTTPS 下載就當安全
2. **CRC 當成簽章使用** — CRC 只防隨機 error，不防蓄意篡改
3. **JTAG/SWD 在量產 firmware 中未禁用** — 攻擊者可以 dump firmware、修改記憶體
4. **輸入驗證不足** — Modbus parser 不檢查 function code / register range
5. **無 anti-rollback** — 攻擊者可降級到有已知漏洞的舊版本

## 7. 小結

- FR3 = 系統沒被偷改：Secure Boot → FW 簽章 → 輸入驗證 → 惡意碼防護
- 信任必須有起點 (Root of Trust)：硬體 ROM → bootloader → kernel → app
- CRC ≠ 安全簽章：CRC 防隨機、HMAC/簽章防蓄意
- 量產後 JTAG 必須鎖——不然攻擊者物理接觸就能全拿

## 8. 下一篇

> [FR 4 (DC)：資料機密性與金鑰管理](05-fr4-data-confidentiality.md)

系統沒被改（FR3）——但機密資料有沒有被偷看？

---

相關：[CONTEXT.md](../../CONTEXT.md)、[IEC 62443-4-2 官方頁](https://webstore.iec.ch/en/publication/34421)


---

## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| **CR** | Component Requirement | 組件安全需求，IEC 62443-4-2 定義 |
| **CSA** | Component Security Assurance | ISASecure 組件安全認證 |
| **DC** | Data Confidentiality | 資料機密性 (FR4) |
| **ED** | Embedded Device | 嵌入式裝置組件 (IEC 62443-4-2 組件類型) |
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| **FW** | Firmware | 韌體，嵌入式裝置上的軟體 |
| **HD** | Host Device | 主機裝置組件 (IEC 62443-4-2 組件類型) |
| **HMAC** | Hash-based Message Authentication Code | 雜湊訊息鑑別碼，驗證完整性+來源 |
| **ISASecure** | ISA Security Compliance Institute | ISA 資安合規協會，營運 IEC 62443 認證方案 |
| **JTAG** | Joint Test Action Group | 聯合測試行動組，晶片除錯介面標準 |
| **MMU** | Memory Management Unit | 記憶體管理單元，虛擬記憶體映射 |
| **MPU** | Memory Protection Unit | 記憶體保護單元，MCU 層級的記憶體隔離 |
| **ND** | Network Device | 網路裝置組件 (IEC 62443-4-2 組件類型) |
| **OS** | Operating System | 作業系統 |
| **ROP** | Return-Oriented Programming | 返回導向程式設計，繞過 NX/DEP 的利用技術 |
| **SA** | Software Application | 軟體應用組件 (IEC 62443-4-2 組件類型) |
| **SHA** | Secure Hash Algorithm | 安全雜湊演算法，產生固定長度摘要 |
| **SI** | System Integrity | 系統完整性 (FR3) |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| **SWD** | Serial Wire Debug | 序列線除錯，ARM MCU 的除錯介面 |
| **UART** | Universal Asynchronous Receiver-Transmitter | 通用非同步收發器，序列通訊 |
| **UC** | Use Control | 使用控制 (FR2) |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
