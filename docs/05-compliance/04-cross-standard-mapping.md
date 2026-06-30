# 與其他標準的對照 — ISO 27001 / NIST CSF / Common Criteria

> 一句話定位：IEC 62443 不是真空中的標準。在工控資安的合規場景中，客戶可能同時要求 ISO 27001、NIST CSF、或 Common Criteria。本篇回答：這些標準之間的關係是什麼？什麼時候用哪個？彼此的覆蓋範圍如何對照？
>
> 前置：[常見合規落差](03-gap-analysis.md)

## 1. 四個標準的定位差異

| | IEC 62443 | ISO 27001 | NIST CSF | Common Criteria (ISO 15408) |
|---|---|---|---|---|
| **類型** | 工控專用標準 | 通用 ISMS（資訊安全管理系統） | 資安框架（非標準） | 產品安全評估標準 |
| **範圍** | IACS（工控系統）全生命週期 | 任何組織的資訊安全管理 | 任何組織的資安成熟度評估 | 單一 IT 產品的 security target |
| **對象** | Asset Owner + Integrator + Product Supplier | 任何組織（不限產業） | 任何組織（特別是美國聯邦） | 產品開發商 |
| **證書** | ISASecure SDLA/CSA/SSA | ISO 27001 認證 | 無證書（自評框架） | CC EAL 1-7 |
| **強制性** | 客戶採購要求（非法律） | 合約/法規要求 | 美國聯邦合約要求 | 政府採購（國防、情報） |

## 2. IEC 62443 vs ISO 27001

### 2.1 本質差異

ISO 27001 是 **ISMS (Information Security Management System)**——管的是「組織怎麼治理資訊安全」，包括 policy、風險評鑑、內部稽核、持續改善。

IEC 62443 是 **IACS security**——管的是「工控系統從設計到退役的技術安全」。它的 4-2 直接對到產品規格，4-3-3 直接對到系統架構。

**交集**：IEC 62443-2-1（業主安全管理計畫）與 ISO 27001 的 ISMS 高度相似——兩者都談 policy、組織、風險管理。但它們不是同等關係——62443-2-1 是 ISMS 在工控領域的特化版。

### 2.2 對照矩陣

| IEC 62443 FR | ISO 27001 Annex A 控制項 |
|---|---|
| FR1 (IAC) | A.9.2 User access management、A.9.4 System and application access control |
| FR2 (UC) | A.9.1 Business requirements of access control、A.9.2.3 Management of privileged access rights |
| FR3 (SI) | A.12.2 Protection from malware、A.14.2 Security in development |
| FR4 (DC) | A.10 Cryptography、A.11 Physical and environmental security |
| FR5 (RDF) | A.13 Communications security、A.13.1 Network security management |
| FR6 (TRE) | A.12.4 Logging and monitoring、A.16 Information security incident management |
| FR7 (RA) | A.17 Information security aspects of business continuity management |
| 4-1 (SDLC) | A.14 System acquisition, development and maintenance（整章） |

> ISO 27001 覆蓋了 FR 概念，但沒有 SL 分級（要做到多強？）、沒有組件分類（哪種裝置適用哪些要求？）、沒有 Zone & Conduit 模型。IEC 62443 多出來的價值是**工控特化 + 可量測 + 可認證到產品層級**。

## 3. IEC 62443 vs NIST CSF

### 3.1 本質差異

NIST CSF (Cybersecurity Framework) 是 NIST 設計給美國聯邦機構與 critical infrastructure 的**自評框架**，不是可認證的標準。五個核心功能：Identify → Protect → Detect → Respond → Recover。

NIST 另有 NIST SP 800-82（工控系統安全指南），那是跟 IEC 62443 更接近的文獻。

### 3.2 對照

| NIST CSF Function | IEC 62443 對應 |
|---|---|
| **Identify** | Zone & Conduit 劃分、風險評估 (-3-2)、資產盤點 |
| **Protect** | FR1-5 (IAC/UC/SI/DC/RDF)、4-2 CR 要求 |
| **Detect** | FR6 (TRE) — 安全事件記錄與監控 |
| **Respond** | P6 (DM) 漏洞處置、FR6 事件回應 |
| **Recover** | FR7 (RA) 資源可用性、fail-safe |

> NIST CSF 給框架語言（「你應該做 Identify」），IEC 62443 給具體技術要求（「你要做 IAC 到 SL-C 3，具體包含 x509 mTLS + SE 保護的私鑰」）。兩者是互補的：CSF 適合跟管理層溝通、62443 適合跟工程團隊溝通。

## 4. IEC 62443 vs Common Criteria (ISO 15408)

### 4.1 本質差異

Common Criteria (CC) 是 IT 產品的通用安全評估標準。它的核心概念：
- **Protection Profile (PP)**：定義一類產品的標準化安全要求（如「智慧卡 PP」「防火牆 PP」）
- **Security Target (ST)**：具體產品的宣稱安全功能
- **EAL 1-7**：評估保證等級（EAL 1 = 功能測試，EAL 7 = 形式化驗證）

IEC 62443 與 CC 的主要差異：
- CC 是通用 IT 產品標準，沒有工控特化
- CC 的 EAL 衡量的是「評估有多深」，不是「產品有多安全」（EAL 1 可以是非常安全的產品但只經過淺層測試）
- IEC 62443 的 SL 衡量的是「產品能防多強的攻擊者」

### 4.2 邊界：什麼時候用 CC、什麼時候用 62443

| 場景 | 用 CC | 用 IEC 62443 |
|---|---|---|
| 智慧卡、HSM、TPM 等密碼產品 | ✓ (有對應 PP) | ✗ |
| 一般 IT 軟體（防火牆、IDS） | ✓ | △ (可選) |
| PLC、RTU、DCS 等工控設備 | ✗ (無對應 PP) | ✓ |
| AMR 控制器、工業 IoT 裝置 | ✗ | ✓ |
| 完整控制系統 | ✗ | ✓ (SSA) |

## 5. 實務建議：客戶要求多個標準時

| 客戶要求 | 策略 |
|---|---|
| IEC 62443 + ISO 27001 | 組織層用 ISO 27001 ISMS（一次滿足管理審查）；產品層用 62443-4-2（一次滿足技術審查）。兩者各有側重、不重複 |
| IEC 62443 + NIST CSF | 用 NIST CSF 跟管理層對話（成熟度語言）；用 62443 跟工程團隊對話（技術規格語言） |
| IEC 62443 + CC | 產品是工控設備 → 只用 62443。產品是通用 IT（如防火牆賣進工控） → 可能需要兩者 |
| 客戶說「符合資安標準」但沒指明哪個 | 問回去：管理面還技術面？組織面還是產品面？—幫他們釐清後推薦對的標準 |

## 6. 小結

- IEC 62443 是工控專用的完整標準（管理+技術+產品），ISO 27001 是通用管理系統
- NIST CSF 是框架語言（搭管理層），62443 是技術語言（搭工程師）
- Common Criteria 是 IT 產品評估，對工控設備沒有對應的 Protection Profile
- 多重標準場景：ISO 27001 管組織、62443-4-2 管產品、NIST CSF 搭高層—三者互補

---

> **知識庫全文完。**
>
> 使用方式：從 [README.md](../../README.md) 依閱讀動線進入各階段的深度篇章。
> 當需要分析現有工控系統時，以本庫的 FR 1-7 矩陣 + SL 模型為框架，逐條對照現況找出 gap。

相關：[CONTEXT.md](../../CONTEXT.md)、[ISO 27001](https://www.iso.org/standard/82875.html)、[NIST CSF](https://www.nist.gov/cyberframework)、[Common Criteria](https://www.iso.org/standard/72891.html)


---

## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| **AMR** | Autonomous Mobile Robot | 自主移動機器人/搬運車 |
| **CC** | Common Criteria | 資訊技術安全評估共同準則 (ISO 15408) |
| **CR** | Component Requirement | 組件安全需求，IEC 62443-4-2 定義 |
| **CSA** | Component Security Assurance | ISASecure 組件安全認證 |
| **CSF** | Cybersecurity Framework | 資安框架 (NIST) |
| **DC** | Data Confidentiality | 資料機密性 (FR4) |
| **DCS** | Distributed Control System | 分散式控制系統 |
| **EAL** | Evaluation Assurance Level | 評估保證等級 (Common Criteria, 1-7) |
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| **HSM** | Hardware Security Module | 硬體安全模組，專用加密金鑰管理硬體 |
| **IAC** | Identification and Authentication Control | 識別與鑑別控制 (FR1) |
| **IACS** | Industrial Automation and Control System | 工業自動化與控制系統，IEC 62443 的守備範圍 |
| **ISASecure** | ISA Security Compliance Institute | ISA 資安合規協會，營運 IEC 62443 認證方案 |
| **NIST** | National Institute of Standards and Technology | 美國國家標準技術研究院 |
| **PLC** | Programmable Logic Controller | 可程式邏輯控制器 |
| **RA** | Resource Availability | 資源可用性 (FR7) |
| **RDF** | Restricted Data Flow | 限制資料流 (FR5) |
| **RTU** | Remote Terminal Unit | 遠端終端單元 |
| **SDLA** | Secure Development Lifecycle Assurance | ISASecure 安全開發流程認證 |
| **SDLC** | Secure Development Lifecycle | 安全開發生命週期，IEC 62443-4-1 規範 |
| **SE** | Secure Element | 安全元件，晶片級金鑰儲存 (如 ATECC608) |
| **SI** | System Integrity | 系統完整性 (FR3) |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| **SSA** | System Security Assurance | ISASecure 系統安全認證 |
| **TPM** | Trusted Platform Module | 可信平台模組，平台身分與金鑰儲存 |
| **TRE** | Timely Response to Events | 事件及時回應 (FR6) |
| **UC** | Use Control | 使用控制 (FR2) |
| **mTLS** | Mutual TLS | 雙向 TLS，雙方皆以憑證互相鑑別 |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
