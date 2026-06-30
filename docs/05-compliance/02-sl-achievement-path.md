# SL 達成路徑 — 從風險評估到合規驗證的完整閉環

> 一句話定位：本篇把 Zone & Conduit、SL-T/SL-C/SL-A、4-1 SDLC、4-2 CR 這四個之前在各自篇章散開的概念串成一條完整閉環——從一個工廠業主決定「我要保護這個系統」到最終確認「買回來的東西真的達到目標」的完整路徑。
>
> 前置：[Security Levels](../01-foundations/03-security-levels.md)、[ISASecure 認證](01-isasecure-certification.md)
> 下一篇：[常見合規落差與補強策略](03-gap-analysis.md)

## 1. 完整閉環：六步驟

```
步驟 1         步驟 2           步驟 3          步驟 4         步驟 5         步驟 6
定義 SUC  →  劃分 Zone &  →  風險評估定  →  挑選組件    →  部署驗收    →  營運監控
            Conduit        SL-T           (SL-C≥SL-T)    (SL-A≥SL-T)
──────────    ──────────      ──────────      ──────────    ──────────    ──────────
Asset Owner  Asset Owner    Asset Owner    System Integ   System Integ  Asset Owner
                                          + Product Sup
```

### 2. 步驟 1：定義 SUC (System Under Consideration)

不要一次評估全廠。定義這次評估的**範圍邊界**：

- 哪些設備、在哪個網段、負責什麼功能
- 邊界內包含哪些 Zone、哪些 Conduit
- **明確畫出「評估邊界外」的部分**（例如 Internet 不在範圍內，但 Firewall 在）

產出：`SUC 定義文件`（一張架構圖 + 資產清單）

### 3. 步驟 2：劃分 Zone & Conduit

見 [Zone & Conduit 篇](../01-foundations/02-zone-and-conduit.md)。

典型產出：ISA-95 Purdue Model 分層圖（L0-L4），每個 Zone 標示：
- 資產清單
- 通訊協定與 port
- 信賴等級（哪些設備在同一個 VLAN）
- Conduit 連線需求

### 4. 步驟 3：風險評估 → 決定 SL-T

![SL-T 決定流程：風險矩陣 × 攻擊者模型 × 公司風險承受度]

對每個 Zone 和每個 Conduit，執行以下分析：

#### 4.1 威脅識別

| 威脅來源 | 範例 |
|---|---|
| 外部攻擊者 | Internet 掃描、釣魚郵件、供應鏈漏洞 |
| 內部攻擊者 | 離職員工、被入侵的承包商筆電 |
| 物理攻擊者 | 可進入工廠的訪客/維修商 |
| 環境事件 | 停電、水災、硬體故障 |

#### 4.2 影響評估（每個 Zone）

| Zone | 最壞情況 | 影響等級 |
|---|---|---|
| L1 控制區 | PLC 被控 → 全廠停機/爆炸 | Critical (5/5) |
| L2 監控區 | SCADA 被駭 → 操作員看到假畫面 | High (4/5) |
| L3 營運區 | MES 停擺 → 工單亂發 | Medium (3/5) |
| DMZ | 被當跳板 → 入侵控制區 | High (4/5) |

#### 4.3 攻擊可能性評估

| 攻擊者類型 | 動機 | 能力/資源 | 對應 SL |
|---|---|---|---|
| Script kiddie | 好奇 | 低 | SL 2 |
| Competitor | 經濟利益 | 中 | SL 3 |
| Nation-state | 地緣政治 | 高 | SL 4 |

#### 4.4 決定 SL-T

```
SL-T = max(風險矩陣 × 攻擊者分析)

例：L1 控制區
  - 影響 Critical (5/5) + 預期攻擊者 SL 3 (competitor/APT)
  - → SL-T = 3
```

產出：`每個 Zone/Conduit 的 SL-T 宣告`

### 5. 步驟 4：挑選組件（SL-C ≥ SL-T）

對標的 Zone SL-T，檢查候選產品的 SL-C 矩陣：

| FR | L1 Zone SL-T | 產品 A SL-C | 產品 B SL-C |
|---|---|---|---|
| FR1 (IAC) | 3 | 3 | 2 ⚠ |
| FR2 (UC) | 2 | 3 | 2 |
| FR3 (SI) | 3 | 3 | 3 |
| FR4 (DC) | 2 | 2 | 2 |
| FR5 (RDF) | 2 | 1 ⚠ | 2 |
| FR6 (TRE) | 2 | 2 | 1 ⚠ |
| FR7 (RA) | 2 | 2 | 2 |

產品 A：FR5 SL-C 只有 1 < SL-T 2 → 需要在 Conduit 層補償（CCSC 2）
產品 B：三個 FR 不合格 → 換產品或大幅補償

**選品決策文件**：記錄選了哪個產品、哪些 FR 靠補償、補償方案是什麼。

### 6. 步驟 5：部署與驗收（SL-A ≥ SL-T）

整合完成後，實際測試：
- **安全功能測試**：每個宣稱的安全功能實際測一遍
- **滲透測試**：從外部/內部/物理三個攻擊面試著打進去
- **弱點掃描**：整個系統掃一遍已知 CVE
- **Conduit 測試**：測試跨 Zone 通訊確實只有白名單的規則能通

產出：`SL-A 驗收報告` + `殘餘風險登記簿`

### 7. 步驟 6：營運中的持續監控

合規不是一次性事件。營運中需要：
- 定期弱點掃描（每季）
- 安全事件監控（SIEM/SOC）
- 變更管理：任何架構變更 → 重新評估對 SL-A 的影響
- 定期重新評估 SL-T（新的威脅情報）

## 8. 誰負責什麼

| 角色 | 負責步驟 | 產出 |
|---|---|---|
| **Asset Owner** | 步驟 1, 2, 3, 6 | SUC 文件、Zone 圖、SL-T 宣告、營運監控 |
| **System Integrator** | 步驟 4, 5 | 選品決策、補償方案、SL-A 驗收報告 |
| **Product Supplier** | 事前提供 | SL-C 宣告、hardening guide、安全通告 |

## 9. 小結

- SL-T/C/A 閉環 = 6 步驟，從定義範圍到營運監控
- 關鍵文件：SL-T 宣告、SL-C 矩陣、補償方案、SL-A 驗收報告
- 不要求所有 FR 滿分：CCSC 2 允許系統層補償
- 責任分離：業主設標準、整合商挑產品、供應商宣告能力

## 10. 下一篇

> [常見合規落差與補強策略](03-gap-analysis.md)

閉環知道了。實務上——**最常掉在哪個坑？補強成本怎麼排？**

---

相關：[CONTEXT.md](../../CONTEXT.md)、[IEC 62443-3-2 官方頁](https://webstore.iec.ch/en/publication/30727)


---

## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| **APT** | Advanced Persistent Threat | 進階持續性威脅，國家級/組織化攻擊 |
| **CCSC** | Common Component Security Constraint | 通用組件安全約束，4-2 定義 4 條鐵律 |
| **CR** | Component Requirement | 組件安全需求，IEC 62443-4-2 定義 |
| **CVE** | Common Vulnerabilities and Exposures | 通用漏洞揭露編號 |
| **DC** | Data Confidentiality | 資料機密性 (FR4) |
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| **IAC** | Identification and Authentication Control | 識別與鑑別控制 (FR1) |
| **ISASecure** | ISA Security Compliance Institute | ISA 資安合規協會，營運 IEC 62443 認證方案 |
| **MES** | Manufacturing Execution System | 製造執行系統，管理工單與生產排程 |
| **PLC** | Programmable Logic Controller | 可程式邏輯控制器 |
| **RA** | Resource Availability | 資源可用性 (FR7) |
| **RDF** | Restricted Data Flow | 限制資料流 (FR5) |
| **SCADA** | Supervisory Control and Data Acquisition | 監控與資料擷取系統 |
| **SDLC** | Secure Development Lifecycle | 安全開發生命週期，IEC 62443-4-1 規範 |
| **SI** | System Integrity | 系統完整性 (FR3) |
| **SIEM** | Security Information and Event Management | 資安資訊與事件管理系統 |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-A** | Achieved Security Level | 達成安全等級，部署後經評估確認的實際等級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| **SL-T** | Target Security Level | 目標安全等級，業主經風險評估後設定 |
| **SUC** | System Under Consideration | 評估目標系統，風險評估的範圍邊界 |
| **TRE** | Timely Response to Events | 事件及時回應 (FR6) |
| **UC** | Use Control | 使用控制 (FR2) |
| **VLAN** | Virtual LAN | 虛擬區域網路，邏輯隔離 |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
