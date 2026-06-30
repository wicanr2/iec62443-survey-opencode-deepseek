# 03 組件 FR 逐條解說 (IEC 62443-4-2)

回答「軟硬體組件要滿足哪些技術安全要求」——以 7 條 FR 為骨架，每條展開軟體實作、硬體支援、SL 1-4 要求差異、常見不合格項目。

## 閱讀順序

| # | 檔案 | 一句話 |
|---|---|---|
| 0 | [01-component-classification.md](01-component-classification.md) | **先讀這篇**：四類組件（SA/ED/HD/ND）+ CCSC 1-4 通用約束 |
| 1 | [02-fr1-identification-authentication.md](02-fr1-identification-authentication.md) | FR1 (IAC)：你是誰？人/程序/裝置的鑑別 |
| 2 | [03-fr2-use-control.md](03-fr2-use-control.md) | FR2 (UC)：你能做什麼？RBAC + 最小權限 |
| 3 | [04-fr3-system-integrity.md](04-fr3-system-integrity.md) | FR3 (SI)：被偷改了嗎？Secure Boot + 簽章 |
| 4 | [05-fr4-data-confidentiality.md](05-fr4-data-confidentiality.md) | FR4 (DC)：被偷看了嗎？TLS + 金鑰管理 |
| 5 | [06-fr5-restricted-data-flow.md](06-fr5-restricted-data-flow.md) | FR5 (RDF)：誰能跟誰通？Default deny + 分區 |
| 6 | [07-fr6-timely-response.md](07-fr6-timely-response.md) | FR6 (TRE)：出事知道嗎？記錄 + 告警 + 稽核 |
| 7 | [08-fr7-resource-availability.md](08-fr7-resource-availability.md) | FR7 (RA)：還能運作嗎？抗 DoS + fail-safe |


---

## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| **CCSC** | Common Component Security Constraint | 通用組件安全約束，4-2 定義 4 條鐵律 |
| **DC** | Data Confidentiality | 資料機密性 (FR4) |
| **DoS** | Denial of Service | 服務阻斷攻擊，耗盡系統資源使其無法回應 |
| **ED** | Embedded Device | 嵌入式裝置組件 (IEC 62443-4-2 組件類型) |
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| **HD** | Host Device | 主機裝置組件 (IEC 62443-4-2 組件類型) |
| **IAC** | Identification and Authentication Control | 識別與鑑別控制 (FR1) |
| **ND** | Network Device | 網路裝置組件 (IEC 62443-4-2 組件類型) |
| **RA** | Resource Availability | 資源可用性 (FR7) |
| **RBAC** | Role-Based Access Control | 基於角色的存取控制 |
| **RDF** | Restricted Data Flow | 限制資料流 (FR5) |
| **SA** | Software Application | 軟體應用組件 (IEC 62443-4-2 組件類型) |
| **SI** | System Integrity | 系統完整性 (FR3) |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **TLS** | Transport Layer Security | 傳輸層安全協定，加密通訊 |
| **TRE** | Timely Response to Events | 事件及時回應 (FR6) |
| **UC** | Use Control | 使用控制 (FR2) |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
