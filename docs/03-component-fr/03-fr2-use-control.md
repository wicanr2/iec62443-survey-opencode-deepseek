# FR 2 (UC) — 使用控制與授權

鑑別完「你是誰」(FR1) 之後，下一題是「你能做什麼」(FR2)。使用控制 = 授權 + 最小權限 + 職責分離 + 不可否認性。在工控系統中，授權錯誤不只造成資料損失，而是可能導致安全停機功能被關掉。

下一篇：[→ FR 3 (SI)：系統完整性](04-fr3-system-integrity.md)

## 1. 根本問題

一個實習生不該有權力停止全廠的 PLC。一個維修人員不該能查看生產配方。一個被 malware 感染的 MES 系統不該能對 AMR 車隊發緊急停止指令。

FR2 的核心鐵律：最小權限 (Least Privilege)

但最小權限在工控場景有特殊困難：

| 困難 | IT 解法 | OT 的額外問題 |
|---|---|---|
| 角色模糊 | RBAC | 同一人早上是操作員、下午是維修員、半夜是值班主管——角色隨 shift 變化 |
| 權限動態 | 靜態 RBAC | 正常運轉時鎖住一切變更、維修模式時開放——權限隨產線狀態變化 |
| 過度限制 = 危險 | 拒絕存取 → 叫 IT | 拒絕存取 → 緊急停止按鈕無效 → 可能死人 |

> FR2 在 OT 的核心矛盾：授權太嚴 = 影響可用性/安全；授權太鬆 = 安全漏洞。 解法是 role engineering——在設計階段就做好角色分析，不是事後補。

## 2. SL-C 1-4 的要求差異

> 本 FR 對應 IEC 62443-4-2 中的 13 條 CR（CR 2.1 – CR 2.13）。每條 CR 對特定 SL-C 等級有要求，下表為各等級的綜合摘要。CR 條號與詳細要求請參閱標準原文或 ISASecure CSA-311。
>
> [¹]: 同上

| SL | 要求 |
|---|---|
| **SL-C 1** | 基本授權（區分 admin vs user 兩類） |
| **SL-C 2** | RBAC（多角色、可定義權限）；最小權限；工作階段逾時自動鎖定 |
| **SL-C 3** | 職責分離 (SoD)：一個人不能同時發起+核准關鍵操作；含稽核軌跡的使用控制 |
| **SL-C 4** | 動態授權（依環境/時間/系統狀態調整）；雙人授權 (two-person rule) |

### 2.1 職責分離 (SoD) 示範

| 操作 | 需要角色 |
|---|---|
| 建立新的搬運任務 | Operator |
| 修改站點地圖 | Engineer |
| 刪除稽核日誌 | Auditor (且需另一 Auditor 核准) |
| 變更安全參數（速度上限、急停恢復） | Safety Officer（且需雙人授權） |

## 3. 軟體實作指南

### 3.1 RBAC 模型設計

最少三種基礎角色：

| 角色 | 典型權限 |
|---|---|
| Operator | 執行日常操作（啟動/停止任務、監看狀態）。不能改設定、不能改 firmware |
| Engineer | 設定參數、建立/編輯地圖、設定通訊。不能刪除稽核日誌、不能改使用者帳號 |
| Security Admin | 管理使用者、設定角色權限、檢視/匯出稽核日誌。不能執行操作任務 |

進階（SL-C 3+）時增加 Auditor 和 Safety Officer。

### 3.2 工作階段管理

```
要求:
- 閒置 timeout: 15 分鐘鎖定（可設定）
- 最大同時 session 數: 可設定（防 session 濫用）
- Session lock: 解鎖需重新認證（不是點一下恢復）
- 強制登出: admin 可遠端強制登出特定 session
```

### 3.3 「DevMode 全繞過」是 FR2 最常見的致命漏洞

```
if (isDevMode) {
 // 跳過所有權限檢查 ← 這行會在 production 環境留著
 return allowAll();
}
```

> 解決方案：DevMode / debug bypass 必須在 release build 時 compile out，不能是 runtime flag。這是 IEC 62443-4-2 審查中 FR2 最常見的 fail case——其跟 FR3 (SI) 交叉：一個 flag 就繞過所有安全控制 = 完整性失效。

## 4. 硬體支援需求

| 硬體功能 | 說明 |
|---|---|
| Memory Protection Unit (MPU) | 將 firmware 模組隔離在不同 memory region，安全相關模組（如 auth module）不能被他模組寫入 |
| TrustZone / secure world | 授權邏輯放在 TrustZone secure world，normal world code 無法修改授權規則 |
| External watchdog | 獨立監控授權模組的 health——若授權模組 crash，watchdog 觸發 fail-safe |

## 5. 組件類型特化要點

| 類型 | UC 重點 |
|---|---|
| **SA** | 使用者/API 的權限對照表；RBAC model 設計；倚賴 OS 的 user/group |
| **ED** | 授權邏輯內建在 firmware；可能沒有完整的使用者管理界面—透過 web console 或 central mgmt |
| **HD** | 可基於 OS 的 DAC/MAC（如 SELinux）+ application-level RBAC 兩層 |
| **ND** | 管理介面的角色授權（誰可以改 ACL、誰只能看）；通常不需要複雜的 operational RBAC |


- FR2 = 知道你是誰之後，決定你能做什麼
- 核心鐵律：最小權限（Least Privilege）+ 職責分離 (SoD)
- 最大陷阱：DevMode / debug flag 全繞過——必須 compile out，不能是 runtime config
- 工控的特殊難度：授權太嚴 = 影響安全/可用性；需要 role engineering 而非 copy-paste IT RBAC

授權對了（FR2）。下一個問題：系統的 code 本身有沒有被偷改過？ Secure Boot、firmware 簽章、輸入驗證。

---



## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| ACL | Access Control List | 存取控制清單，定義誰能存取什麼資源 |
| AMR | Autonomous Mobile Robot | 自主移動機器人/搬運車 |
| **CR** | Component Requirement | 組件安全需求，IEC 62443-4-2 定義 |
| CSA | Component Security Assurance | ISASecure 組件安全認證 |
| DAC | Discretionary Access Control | 自由選定存取控制，如 Unix permission |
| **ED** | Embedded Device | 嵌入式裝置組件 (IEC 62443-4-2 組件類型) |
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| **HD** | Host Device | 主機裝置組件 (IEC 62443-4-2 組件類型) |
| **IAC** | Identification and Authentication Control | 識別與鑑別控制 (FR1) |
| ISASecure | ISA Security Compliance Institute | ISA 資安合規協會，營運 IEC 62443 認證方案 |
| MAC | Mandatory Access Control | 強制存取控制，如 SELinux |
| MES | Manufacturing Execution System | 製造執行系統，管理工單與生產排程 |
| MPU | Memory Protection Unit | 記憶體保護單元，MCU 層級的記憶體隔離 |
| **ND** | Network Device | 網路裝置組件 (IEC 62443-4-2 組件類型) |
| OS | Operating System | 作業系統 |
| PLC | Programmable Logic Controller | 可程式邏輯控制器 |
| RBAC | Role-Based Access Control | 基於角色的存取控制 |
| **SA** | Software Application | 軟體應用組件 (IEC 62443-4-2 組件類型) |
| SI | System Integrity | 系統完整性 (FR3) |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| SoD | Separation of Duties | 職責分離，一人不能同時發起+核准關鍵操作 |
| **UC** | Use Control | 使用控制 (FR2) |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
