# Practice 1-2：開發管理與安全需求定義

P1 (SM) 是骨架——建立安全開發的組織結構與資源；P2 (SPR) 是起點——把安全需求從威脅分析中逼出來，不是憑空想像。這兩條 Practice 回答了「誰負責」和「要做什麼」。

下一篇：[→ Practice 3-4：安全設計與安全實作](03-secure-design-implementation.md)

## 1. P1 (SM) — 安全管理 (Security Management)

### 1.1 根本問題

開發團隊裡有人在意安全嗎？還是安全是 QA 的事、QA 覺得是 RD 的事、RD 覺得是 IT 的事——最後沒人負責？

P1 回答的問題：
- 安全開發的政策存在嗎？誰寫的？誰簽的？
- 角色定義：誰負責 threat modeling？誰負責 code review？誰收漏洞通報？
- 安全能力：開發者受過安全訓練嗎？知道 OWASP Top 10 嗎？知道工控協定的安全陷阱嗎？
- 流程誰維護？誰負責當有新漏洞類型出現時更新開發流程？

### 1.2 Practice 1 的核心要求

| 要求 | 說明 | ML 差異 |
|---|---|---|
| **安全政策** | 組織要有書面的安全開發政策，由管理層簽署 | ML 1：ad hoc；ML 2+：書面政策 |
| **角色與責任** | 指定安全開發中的各角色（安全負責人、安全 champion、開發者、測試者）及其責任 | ML 2+：角色定義並溝通；ML 3+：角色在全組織標準化 |
| **人員能力** | 開發者、測試者、PM 接受適切的安全訓練 | ML 1：無要求；ML 2：訓練存在；ML 3+：訓練內容與角色對齊、有記錄 |
| **流程管理** | 安全開發流程文件化、版本控制、定期審查 | ML 2：有流程文件；ML 3：流程已演練；ML 4：流程受度量監控 |
| **工具與資源** | 提供安全開發所需工具（靜態分析、fuzz 框架等）與時間 | ML 2+：工具可用；ML 3+：工具使用有記錄 |

### 1.3 工控軟體開發的 P1 陷阱

| 陷阱 | 說明 |
|---|---|
| 安全是 IT 的事 | 嵌入式 firmware 開發者常認為「安全是上層網路的事，我們在底層、不用管」——但 PLC firmware 的漏洞正是 Stuxnet 的入口 |
| **一個人扛全部** | 小團隊沒有專職安全人員 → PM 兼安全負責人 → 合規審查時拿不出證據 |
| 訓練缺工控 context | 一般 OWASP Top 10 訓練不夠——開發者需要知道 Modbus 無認證、CAN bus 無加密、工業協定的 common weakness |

> **實務建議**：即使團隊只有 5 人，也要指定一人為安全聯絡人（不用全職，但要有名有姓有時間），並把威脅建模和安全編碼規範寫成文件——這是 ML 2 的最低門檻。

## 2. P2 (SPR) — 安全需求規格 (Security Requirements)

### 2.1 根本問題

你怎麼知道產品需要什麼安全功能？最常見的錯誤：從功能需求倒推安全需求。 「我們產品要做這個功能 → 順便加個登入好了。」這是安全需求最壞的產生方式。

正確的起點：**威脅分析。** 問「誰會攻擊我們、為什麼、從哪進來」，再推導需要什麼防護。

### 2.2 Practice 2 的核心要求

| 要求 | 說明 | ML 差異 |
|---|---|---|
| **威脅模型** | 每個產品/發行版至少做一次威脅分析，識別威脅與攻擊向量 | ML 2：有做；ML 3+：標準化方法 (如 STRIDE)、定期更新 |
| **安全需求規格** | 從威脅模型導出具體、可測試的安全需求，寫入產品規格 | ML 2：有安全需求文件；ML 3+：需求可追溯至威脅 |
| 法規/標準合規 | 識別產品需要遵循的資安法規與標準 (如 IEC 62443-4-2) | ML 2：有識別；ML 3+：合規矩陣 |
| **安全等級宣告** | 宣告產品對每個 FR 能達到的 SL-C（見基礎篇 03、04） | ML 2：宣告存在；ML 3+：宣告有佐證 |

### 2.3 從威脅模型到安全需求的示範

假設開發一台 AMR 車載控制器（Embedded Device），威脅分析：

| 威脅 | 攻擊者 | 攻擊向量 | 對應 FR | 安全需求 |
|---|---|---|---|---|
| 篡改導航指令，讓車撞牆 | 內部人員/已入侵內網者 | 網路封包注入 (無認證的 API) | FR1 (IAC) | 所有 API 強制 TLS + 裝置憑證雙向認證 |
| 韌體被換成惡意版本 | 物理接觸者 (維修時) | USB/SD 卡更新，無簽章驗證 | FR3 (SI) | 韌體更新必須 ED25519 簽章驗證，bootloader 強制 |
| 竊聽車隊通訊取得路徑 | 鄰近 WiFi 竊聽者 | HTTP 明文傳輸 | FR4 (DC) | 全鏈路 TLS 1.3 |
| MES 系統被入侵後下假工單 | APT | 橫向移動至車隊控制網路 | FR5 (RDF) | 車隊控制區與 MES 區之間只允許 whitelist IP + Modbus DPI |

> 重點：安全需求不是「想出來的」，是「從威脅推導出來的」。 每一條安全需求必須能回答「這個需求防住哪個威脅」。

### 2.4 P2 與 FR/SL 的對接

P2 的產出直接決定了產品在 4-2 中的 SL-C 宣告。一個典型的安全需求規格文件結構：

安全需求規格文件結構範例（AMR 車載控制器 v2.1）：

1. **威脅模型摘要** (STRIDE, 見 Practice 3)
2. 各 FR 安全需求：
 - FR1 (IAC)：TLS mutual auth + x509 裝置憑證。目標 SL-C 3
 - FR2 (UC) ：RBAC (operator / engineer / auditor)。目標 SL-C 2
 - FR3 (SI) ：Secure Boot + FW 簽章驗證。目標 SL-C 3
 - FR4 (DC) ：全鏈路 TLS 1.3。目標 SL-C 2
 - FR5 (RDF)：VLAN + service ACL。目標 SL-C 2
 - FR6 (TRE)：Syslog 輸出 + 安全事件記錄保留 ≥30 天。目標 SL-C 2
 - FR7 (RA) ：fail-safe（斷線自動減速停車）。目標 SL-C 2
3. **補償對策**：本組件不支援 L7 DPI（FR5 由 Conduit 補償）


- P1 (SM)：建立安全開發的組織條件——政策、角色、能力、工具。沒有這層，後面的 Practices 都是 air
- P2 (SPR)：從威脅模型出發，導出具體安全需求（不是臆想，是證據驅動）
- P1+P2 構成了整個 SDLC 的輸入：P2 定義「要做什麼安全」，P3-P5 負責「做出來並驗證」

威脅模型有了（P2），安全需求定義了（P2），下一步：安全架構設計 + 安全編碼。

---



## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| ACL | Access Control List | 存取控制清單，定義誰能存取什麼資源 |
| AMR | Autonomous Mobile Robot | 自主移動機器人/搬運車 |
| APT | Advanced Persistent Threat | 進階持續性威脅，國家級/組織化攻擊 |
| DC | Data Confidentiality | 資料機密性 (FR4) |
| DPI | Deep Packet Inspection | 深層封包檢測，辨識應用層協定內容 |
| **ED** | Embedded Device | 嵌入式裝置組件 (IEC 62443-4-2 組件類型) |
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| FW | Firmware | 韌體，嵌入式裝置上的軟體 |
| **IAC** | Identification and Authentication Control | 識別與鑑別控制 (FR1) |
| MES | Manufacturing Execution System | 製造執行系統，管理工單與生產排程 |
| **ML** | Maturity Level | 成熟度等級，IEC 62443-4-1 對開發流程的分級 (1-4) |
| PLC | Programmable Logic Controller | 可程式邏輯控制器 |
| **RA** | Resource Availability | 資源可用性 (FR7) |
| RBAC | Role-Based Access Control | 基於角色的存取控制 |
| **RDF** | Restricted Data Flow | 限制資料流 (FR5) |
| SDLC | Secure Development Lifecycle | 安全開發生命週期，IEC 62443-4-1 規範 |
| SI | System Integrity | 系統完整性 (FR3) |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| **SPR** | Security Practice: Requirements | 本庫自訂代號：安全需求規格 (P2) |
| STRIDE | Spoofing/Tampering/Repudiation/InfoDisclosure/DoS/Elevation | 微軟六面向威脅建模方法 |
| TLS | Transport Layer Security | 傳輸層安全協定，加密通訊 |
| **TRE** | Timely Response to Events | 事件及時回應 (FR6) |
| **UC** | Use Control | 使用控制 (FR2) |
| VLAN | Virtual LAN | 虛擬區域網路，邏輯隔離 |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
