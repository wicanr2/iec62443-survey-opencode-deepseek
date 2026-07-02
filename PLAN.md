# IEC 62443 軟硬體安全規範知識庫 — 建設計畫

## 專案定位

為工程團隊打造一份「從根本問題推導、圖文並茂、繁體中文」的 IEC 62443 軟硬體安全合規知識庫。
聚焦 IEC 62443-4-2（組件技術安全要求）與 IEC 62443-4-1（安全產品開發生命週期），輔以 IEC 62443-3-3（系統層安全要求）及 -3-2（Zone & Conduit / SL-T）作為上下文。

**一句話說**：回答「軟硬體產品要拿到 IEC 62443 認證，我該做什麼、為什麼這樣做、東西要長成怎樣」。

---

## 目錄結構

```
iec62443-kb/
├── README.md          # 索引入口 + 零起點導言 + 閱讀動線
├── PLAN.md            # 本檔：分輪計畫與進度表
├── CONTEXT.md         # 術語表（ubiquitous language）
├── img/               # 所有 SVG 圖（白底、Orange 點綴、CJK 字型）
└── docs/
    ├── 01-foundations/    # 基礎概念（4 篇）
    ├── 02-sdlc/           # 安全開發生命週期（6 篇）
    ├── 03-component-fr/   # 組件 FR 1-7 逐條解說（8 篇）
    ├── 04-hardware/       # 硬體安全專題（3 篇）
    └── 05-compliance/     # 合規與認證實務（4 篇）
總計 25 篇
```

---

## 分輪計畫（每輪一主題，研究 → 書寫 → 配圖 → 接索引 → push → 審查）

### Round 1–4：基礎概念（docs/01-foundations/）

| R | 檔名 | 標題 | 核心問題 | 狀態 |
|---|---|---|---|---|
| 1 | `01-iec62443-overview.md` | IEC 62443 全景圖 — 為什麼 OT 需要獨立的資安標準 | IT vs OT：CIA triad 為什麼在 OT 反過來是 AIC？工控系統的 safety / availability 為什麼排第一？IEC 62443 標準族全景地圖 | ✅ 2026-06-30 |
| 2 | `02-zone-and-conduit.md` | Zone & Conduit — 信任邊界的根本推導 | 為什麼不能整廠打平當一個網路？Zone 分割的物理/邏輯意義；Conduit 的保護角色；defense-in-depth 的幾何視角 | ✅ 2026-06-30 |
| 3 | `03-security-levels.md` | Security Levels (SL) — SL1-SL4 威脅模型與 SL-T/SL-C/SL-A 三角 | 誰會來攻擊你的工控系統？（script kiddie → 國家級）SL 分級根本邏輯：靠對手的能力/資源/動機決定，不是靠技術規格 | ✅ 2026-06-30 |
| 4 | `04-foundational-requirements.md` | FR 1-7 全景 — 七個基礎安全需求的由來 | 從 AIC + Safety 的根本矛盾推導出七個 FR（IAC/UC/SI/DC/RDF/TRE/RA），解釋「為什麼是這七個」而非「只是背條文」 | ✅ 2026-06-30 |

### Round 5–10：安全開發生命週期（docs/02-sdlc/）

| R | 檔名 | 標題 | 核心問題 | 狀態 |
|---|---|---|---|---|
| 5 | `01-secure-sdlc-overview.md` | Secure SDLC 全景 — IEC 62443-4-1 的軟體開發流程要求 | 八個 Practice 的整體架構；ML 1-4 Maturity Level；4-1 在認證生態中的角色（SDLA 方案） | ✅ 2026-06-30 |
| 6 | `02-security-management-requirements.md` | Practice 1-2：開發管理與安全需求定義 | 安全組織建立、角色、訓練、威脅模型 → 安全需求規格 | ✅ 2026-06-30 |
| 7 | `03-secure-design-implementation.md` | Practice 3-4：安全設計與安全實作 | STRIDE threat modeling for embedded；安全編碼規範、SAST、code review、相依掃描 | ✅ 2026-06-30 |
| 8 | `04-security-testing-vnv.md` | Practice 5-6：安全測試與漏洞管理 | Fuzz testing 工業協定；penetration testing；CVE 處理流程、severity × SLA | ✅ 2026-06-30 |
| 9 | `05-update-patch-management.md` | Practice 7-8：更新發布與安全文件化 | 簽章 + 反回滾 + 安全更新信任鏈；hardening guide、安全功能說明書 | ✅ 2026-06-30 |
| 10 | `06-maturity-levels.md` | Maturity Level (ML) — 從初創到持續改善 | ML 1-4 的具體指標與 per-practice 矩陣；ML 提升路徑；ML vs CMMI 差異 | ✅ 2026-06-30 |

### Round 11–18：組件 FR 1-7 逐條解說（docs/03-component-fr/）

| R | 檔名 | 標題 | 核心問題 | 狀態 |
|---|---|---|---|---|
| 11 | `01-component-classification.md` | 組件分類全景 — 四類組件的邊界與 CCSC | Embedded Device / Host Device / Network Device / Software Application 怎麼分；CCSC 1-4 全文 | ✅ 2026-06-30 |
| 12 | `02-fr1-identification-authentication.md` | FR 1 (IAC)：識別與鑑別控制 | 三種主體（人/程序/裝置）的鑑別；憑證部署流程；HSM/TPM/SE/PUF 選用 | ✅ 2026-06-30 |
| 13 | `03-fr2-use-control.md` | FR 2 (UC)：使用控制與授權 | RBAC 模型設計、職責分離 (SoD)、DevMode 致命陷阱、session management | ✅ 2026-06-30 |
| 14 | `04-fr3-system-integrity.md` | FR 3 (SI)：系統完整性 | Secure Boot 信任鏈、FW 簽章驗證、輸入驗證、anti-rollback、JTAG lock | ✅ 2026-06-30 |
| 15 | `05-fr4-data-confidentiality.md` | FR 4 (DC)：資料機密性與金鑰管理 | TLS/IPsec 實務、靜態加密、金鑰分層 (Root→KEK→DEK)、硬體加速 AES | ✅ 2026-06-30 |
| 16 | `06-fr5-restricted-data-flow.md` | FR 5 (RDF)：限制資料流 | Default deny、服務最小化、通訊矩陣、硬體隔離/Data diode | ✅ 2026-06-30 |
| 17 | `07-fr6-timely-response.md` | FR 6 (TRE)：事件回應與稽核 | 安全事件記錄、syslog 格式、NTP 同步、不可否認性、嵌入式儲存限制 | ✅ 2026-06-30 |
| 18 | `08-fr7-resource-availability.md` | FR 7 (RA)：資源可用性 | 抗 DoS、資源管理（安全功能不降級）、fail-safe design、watchdog、斷電恢復 | ✅ 2026-06-30 |

### Round 19–21：硬體安全專題（docs/04-hardware/）

| R | 檔名 | 標題 | 核心問題 | 狀態 |
|---|---|---|---|---|
| 19 | `01-hardware-root-of-trust.md` | 硬體信任根與 Secure Boot 鏈 | 信任鏈 ROM → BL → OS → App；OTP/efuse 公鑰儲存；A/B partition | ✅ 2026-06-30 |
| 20 | `02-physical-tamper-protection.md` | 物理防篡改與除錯埠管理 | 機殼開關/導電網格/環境感測器；tamper response (zeroize keys)；JTAG/SWD 生命週期 | ✅ 2026-06-30 |
| 21 | `03-hsm-secure-element.md` | HSM / TPM / Secure Element 選用指南 | SE 實務整合 (ATECC608)；TPM PCR/attestation；金鑰永不離開安全晶片 | ✅ 2026-06-30 |

### Round 22–25：合規與認證實務（docs/05-compliance/）

| R | 檔名 | 標題 | 核心問題 | 狀態 |
|---|---|---|---|---|
| 22 | `01-isasecure-certification.md` | ISASecure 認證體系 | SDLA/CSA/SSA/ICSA 四方案；認證流程（6-12月，20k-150kUSD）；文件準備最常見退件點 | ✅ 2026-06-30 |
| 23 | `02-sl-achievement-path.md` | SL 達成路徑 — 完整閉環 | SUC→Zone→SL-T→選組件(SL-C)→部署(SL-A)→營運監控，六步驟＋角色分工 | ✅ 2026-06-30 |
| 24 | `03-gap-analysis.md` | 常見合規落差與補強策略 | FR×cost 矩陣；Quick Wins (P0)→TLS/RBAC(P1)→Secure Boot/SE(P2)；長期 roadmap | ✅ 2026-06-30 |
| 25 | `04-cross-standard-mapping.md` | 與其他標準的對照 | IEC 62443 vs ISO 27001 / NIST CSF / Common Criteria；多標準場景的策略建議 | ✅ 2026-06-30 |

### Round 26–29：超出核心 — 系統/業主/IIoT（docs/06-beyond-4x/）

> 觸發：完整標準族遺漏審查（2026-07-02）。核心章刻意聚焦 4-1/4-2；本組把視野擴到相鄰 Part，並明示範圍邊界。

| R | 檔名 | 標題 | 核心問題 | 狀態 |
|---|---|---|---|---|
| 26 | `01-system-requirements-3-3.md` | 系統安全要求 SR — 4-2 的系統層孿生兄弟 | SR vs CR 為何分兩層；完整 51 條 SR；RE 如何隨 SL 加嚴；SSA 認證 | ✅ 2026-07-02 |
| 27 | `02-risk-assessment-3-2.md` | 風險評估 ZCR — SL-T 從哪裡來 | ZCR 七步驟 + ZCR5 十三子步；SL-T 從風險矩陣推導；接回 3-3/4-2 | ✅ 2026-07-02 |
| 28 | `03-asset-owner-program-2-1.md` | 業主安全計畫 — IEC 62443-2-1:2024 | 工控版 ISMS；2024 三大改（SP/SPE、去 ISO 重複、成熟度）；8 個 SPE；接 2-2 | ✅ 2026-07-02 |
| 29 | `04-iiot-profile-1-6.md` | IIoT 應用 — IEC PAS 62443-1-6:2025 | PAS 為何存在；IIoT 鬆動哪些假設；ICSA 認證；生態位 | ✅ 2026-07-02 |

### 遺漏審查修補（2026-07-02，第一級正確性）

| 項目 | 內容 | 狀態 |
|---|---|---|
| 家族全景表 | 補 1-2/1-4/1-6/2-2/6-1/6-2 + 型態(IS/TR/TS/PAS) + 雙編號提醒（`01-overview` / `CONTEXT` / `img/01`） | ✅ |
| SL 定義錯置 | `-1-3` → `-1-1` 修正 | ✅ |
| 6-x 評估方法學 | ISASecure 篇補 6-1:2024 / 6-2:2025 | ✅ |
| ACSSA | 認證方案補 2024 新方案 | ✅ |
| 4-2 版本 | 補 IEC/ISA 雙編號 + prAA:2026 修訂註記 | ✅ |
| SPR 消歧義 | CONTEXT 新增「縮寫消歧義」表 | ✅ |
| 範圍邊界 | `docs/README` 新增範圍說明 | ✅ |

---

## 每輪收尾流程（from skill）

1. 研究 agent → 產出素材（標來源、標待查證）
2. 主迴圈書寫 → 從根本問題推導、配 SVG
3. 用 Chrome headless 轉 PNG 驗圖（文字不重疊、不截斷）
4. 更新 `README.md` 索引、`CONTEXT.md` 術語、本檔進度
5. `git add -A` → 繁中 commit → push
6. 專家審查 agent + 學生審查 agent → 修訂清單
7. 逐條應對修正、再 push

---

## 審查 Backlog

| ID | 提出輪次 | 審查意見 | 處理狀態 |
|---|---|---|---|
| — | — | — | — |

---

## CONTEXT.md 初始架構（將隨每輪擴充）

```markdown
# 術語表 (Ubiquitous Language)

| 縮寫 | 全稱 | 繁體中文 | 說明 |
|---|---|---|---|
| IACS | Industrial Automation and Control System | 工業自動化與控制系統 | 62443 的守備範圍：SCADA/DCS/PLC/RTU 等 |
| OT | Operational Technology | 營運技術 | 相對於 IT，OT 是管物理世界的（產線、閥門、鍋爐…） |
| FR | Foundational Requirement | 基礎安全需求 | 62443 把安全需求歸納為 7 個基礎類別 |
| CR | Component Requirement | 組件安全需求 | 4-2 對單一組件的技術安全要求 |
| SR | System Requirement | 系統安全需求 | 3-3 對整體系統的安全要求 |
| SL-T / SL-C / SL-A | Security Level (Target/Capability/Achieved) | 安全等級（目標/能力/達成） | 三階段安全等級模型 |
| ML | Maturity Level | 成熟度等級 | 4-1 定義的開發流程成熟度（1-4） |
| CCSC | Common Component Security Constraint | 通用組件安全約束 | 4-2 對所有組件的 4 條共通約束 |
| Zone | — | 區域 | 具共同安全需求的資產群組 |
| Conduit | — | 管道 | 連接兩個 Zone 的通訊通道 |
| SUC | System Under Consideration | 評估目標系統 | 風險評估的範圍邊界 |
| Defense in Depth | — | 縱深防禦 | 多層安全控制，單層被破還有下一層 |
```

---

## 寫作紀律（全知識庫適用）

- **第一性原理**：每個概念先問「要解決什麼根本問題」「為什麼是這個設計」，再講怎麼用
- **SVG 配圖**：數學/流程概念一律白底手繪 SVG，橙色點綴，圓角框
- **繁體中文**：自然語言用繁體中文，程式碼/標準條號保留原文
- **不臆造**：不確定標「待查證」，不發明不存在的標準條號或工具名
- **柵欄原則**：發現奇怪的設計要求時，先重建「當初為什麼非這樣不可」
- **誠實標限制**：4-2 原文付費 → 以 ISASecure 公開文件為主要可驗證來源，點出差異

---

## 第一輪啟動指令

```
Round 1: IEC 62443 全景圖 — 為什麼 OT 需要獨立的資安標準
  研究 agent: 找 IT/OT 差異文獻、Stuxnet 案例、NIST 800-82 導論
  產出: docs/01-foundations/01-iec62443-overview.md + img/01-overview-*.svg
```

---

**建立日期**：2026-06-30
**最後更新**：2026-07-02
**進度**：29/29 輪完成 ✅（原核心 25 + 延伸章節 4）+ 遺漏審查第一級修補
**預計總輪數**：29 輪
