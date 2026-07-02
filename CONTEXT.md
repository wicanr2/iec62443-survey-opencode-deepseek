# CONTEXT — IEC 62443 知識庫術語表

這份知識庫聚焦 **IEC 62443 軟硬體組件安全規範**，術語按標準層級與工程領域分組。

## 標準系列代號

| 縮寫 | 全稱 | 中文 | 說明 |
|---|---|---|---|
| IEC 62443 | Industrial communication networks — Network and system security | 工業通訊網路安全標準系列 | ISA/IEC 共同制定，原名 ISA-99 |
| -1-x | General（概念與術語） | 通用基礎 | 概念模型與 **SL 定義** (-1-1)、術語表 (-1-2, 制定中)、一致性度量 (-1-3, 制定中)、生命週期與案例 (-1-4, 制定中)、profiles 方案 (-1-5, 2023)、IIoT 應用 (-1-6, PAS 2025) |
| -2-x | Policies & Procedures（管理程序） | 安全管理 | 業主安全計畫 (-2-1, 2024 Ed.2)、安全防護評等 (-2-2, 制定中)、補丁管理 (-2-3, 2015)、服務商要求 (-2-4, 2023 Ed.2)、業主實作指引 (-2-5, 制定中) |
| -3-x | System security（系統安全） | 系統層安全 | 安全技術 (-3-1)、風險評估 ZCR (-3-2, 2020)、系統需求 SR+SL (-3-3, 2013) |
| -4-x | Component security（組件安全） | 組件層安全 | 開發生命週期 (-4-1, 2018)、組件技術要求 (-4-2, 2019) |
| -6-x | Evaluation（評估方法學） | 認證評估 | 評估 -2-4 方法 (-6-1, TS 2024)、評估 -4-2 方法 (-6-2, TS 2025) |

> **SL 定義在 -1-1**（非 -1-3）。**雙編號**：`IEC 62443-x-x`（IEC 版）與 `ANSI/ISA-62443-x-x`（ISA 版）平行編號，同一 Part 年份/型態可能不同，引用需標明。

## 核心概念

| 縮寫 | 全稱 | 中文 | 說明 |
|---|---|---|---|
| IACS | Industrial Automation and Control System | 工業自動化與控制系統 | IEC 62443 的守備範圍：SCADA/DCS/PLC/RTU/AMR 控制系統等 |
| OT | Operational Technology | 營運技術 | 相對於 IT（管資訊），OT 管物理世界（產線、閥門、鍋爐、搬運車） |
| CIA → AIC | Confidentiality/Integrity/Availability | 機密性/完整性/可用性 | IT 優先序 CIA；OT 反轉為 **AIC**（可用性/安全第一） |
| FR | Foundational Requirement | 基礎安全需求 | 62443 把安全需求歸納為 7 個基礎類別 (FR1-7) |
| SL | Security Level | 安全等級 | 依攻擊者能力/資源/動機分級的 0-4 級 |
| ML | Maturity Level | 成熟度等級 | 4-1 定義的開發流程成熟度 (ML 1-4) |
| CR | Component Requirement | 組件安全需求 | 4-2 對單一組件的技術安全要求（條號格式 CR 1.1, CR 1.2...） |
| SR | System Requirement | 系統安全需求 | 3-3 對整體系統的安全要求 |

## FR 1-7（Foundational Requirements）

| FR | 縮寫 | 全稱 | 要問的根本問題 |
|---|---|---|---|
| FR 1 | IAC | Identification and Authentication Control | 你是誰？我怎麼確認？ |
| FR 2 | UC | Use Control | 你被允許做什麼？ |
| FR 3 | SI | System Integrity | 程式/資料有沒有被偷改？ |
| FR 4 | DC | Data Confidentiality | 機密資料有沒有被偷看？ |
| FR 5 | RDF | Restricted Data Flow | 誰能跟誰通？ |
| FR 6 | TRE | Timely Response to Events | 出事了我知道嗎？ |
| FR 7 | RA | Resource Availability | 系統被打/自己壞了還能運作嗎？ |

## Security Levels (SL)

| 縮寫 | 全稱 | 中文 | 說明 |
|---|---|---|---|
| SL-T | Target Security Level | 目標安全等級 | Asset owner 經風險評估後為各 Zone 設定的所需等級 |
| SL-C | Capability Security Level | 能力安全等級 | 組件 (4-2) 或系統 (3-3) 能達到的安全等級 |
| SL-A | Achieved Security Level | 達成安全等級 | 部署後經評估確認實際達到的等級 |
| SL 0 | No special protection | 無特殊防護 | 無需求 |
| SL 1 | Against casual or coincidental violation | 防意外/誤觸 | 非蓄意攻擊 |
| SL 2 | Intentional violation with simple means | 防一般駭客 | 低資源、一般技能、低動機 |
| SL 3 | Intentional violation with sophisticated means | 防針對性攻擊 | 中資源、專精技能、中動機 |
| SL 4 | Intentional violation with sophisticated means + extended resources | 防國家級 | 高資源、專精、高動機 |

## 組件分類 (Component Types, 4-2)

| 類型 | 英文 | 定義 |
|---|---|---|
| SA | Software Application | 純軟體，無專用硬體，如組態軟體、historian |
| ED | Embedded Device | 專用嵌入式裝置，直接控制/監控製程 (PLC、AMR 控制器等) |
| HD | Host Device | 通用 OS 裝置 (Windows/Linux)，可承載軟體 |
| ND | Network Device | 網路設備 (Switches、Firewalls、Data Diodes) |

## 共同約束 (CCSC, Common Component Security Constraints)

| CCSC | 內容 |
|---|---|
| CCSC 1 | 組件須考量其所在系統的一般安全特性 |
| CCSC 2 | 組件無法自行滿足的技術需求，可由系統層級補償（須記載） |
| CCSC 3 | 組件須套用最小權限原則 (Least Privilege) |
| CCSC 4 | 組件須由符合 IEC 62443-4-1 的開發流程開發與支援 |

## Zone & Conduit 模型

| 術語 | 說明 |
|---|---|
| SUC (System Under Consideration) | 風險評估的目標範圍邊界 |
| Zone | 具共同安全需求的資產群組，有邏輯或實體邊界 |
| Conduit | 連接兩個 Zone 的受控通訊通道 |
| Defense in Depth | 縱深防禦：多層次安全控制，單層被破還有下一層 |

## 認證方案 (ISASecure)

| 縮寫 | 全稱 | 對應標準 |
|---|---|---|
| SDLA | Secure Development Lifecycle Assurance | IEC 62443-4-1 |
| CSA | Component Security Assurance | IEC 62443-4-2 + 4-1 |
| SSA | System Security Assurance | IEC 62443-3-3 + 4-1 |
| ICSA | IIoT Component Security Assurance | IEC 62443-4-2 + 4-1 (IIoT 擴展) |
| ACSSA | Automation and Control System Security Assurance | IEC 62443-2-1 / -2-4 / -3-3 面向（2024 新增，認業主現場系統） |

> **認證方法學**：認證機構「怎麼評」由 IEC TS 62443-6-1 (2024, 評估 -2-4)、-6-2 (2025, 評估 -4-2) 定義；6-2 等於 CSA 的評分方法。

## 縮寫消歧義（易混淆）

| 縮寫 | 本庫用法 | 官方另一意義 | 如何分辨 |
|---|---|---|---|
| **SPR** | 本庫自訂代號：4-1 的 P2「Security Practice: Requirements」（安全需求規格） | **IEC 62443-2-2 的 Security Protection Rating**（安全防護評等，把 SL 技術等級與 ML 成熟度合成單一評分） | 談 4-1 開發 Practice → 本庫 P2；談 2-2 業主現場評分 → 官方 SPR。**兩者無關**，勿混用 |
| **SR** | 3-3 的 System Requirement（系統安全需求） | 亦常被寫成 P2 的縮寫 | 本庫 P2 一律用 SPR，SR 專指 3-3 系統需求 |

> 2-2 在 ISA 版（TR, 2025）稱 **SPS（Security Protection Scheme）**、IEC 版（制定中）稱 **SPR（Security Protection Rating）**，兩套術語尚未統一。

## 全域慣例

- 標準條號 / 程式碼 / 技術識別符保留原文 (如 `FR 1`、`IAC`、`CR 1.1`)
- 繁體中文行文；術語首次出現譯為中文並保留原文
- 「待查證」標在無法從官方來源驗證的陳述後
- 組件 (component) 統一翻「組件」；補償對策 (compensating countermeasures)；安全等級 (security level)
