# ISASecure 認證體系 — CSA / SDLA / SSA 怎麼走

> 一句話定位：IEC 62443 是標準，ISASecure 是認證方案——回答「我的產品/開發流程/系統要怎麼向客戶證明合規」。本篇說明四種認證方案的 scope、流程、角色、費用概念，以及你該從哪個方案開始。
>
> 前置：[SDLC 全景](../02-sdlc/01-secure-sdlc-overview.md)
> 下一篇：[SL 達成路徑 — 從風險評估到合規驗證的完整閉環](02-sl-achievement-path.md)

## 1. 根本問題：有標準 ≠ 有證明

「我們的產品符合 IEC 62443-4-2」——誰說的？你自己說的（self-declaration）？還是第三方驗證過？

Self-declaration 有信用問題。客戶無法判斷你的宣稱是否合理——尤其是技術深度高的標準（4-2 有破百頁的 technical requirements）。

ISASecure 是 ISA (International Society of Automation) 營運的第三方認證方案，是目前 IEC 62443 領域最具公信力的認證體系。

## 2. 四種認證方案

| 方案 | 認證對象 | 對應標準 | 一句話 |
|---|---|---|---|
| **SDLA** | 開發組織的流程 | IEC 62443-4-1 | 證明「你會用安全的方式開發產品」 |
| **CSA** | 單一組件（產品） | IEC 62443-4-2 + 4-1 | 證明「這個產品安全 + 開發流程安全」 |
| **SSA** | 系統（多組件整合） | IEC 62443-3-3 + 4-1 | 證明「這個系統安全 + 開發流程安全」 |
| **ICSA** | IIoT 組件 | IEC 62443-4-2 + 4-1（IIoT 擴展） | CSA 的 IIoT 特化版 |

### 2.1 方案之間的關係

```
SDLA ────────────────────────────────
  │                                    \
  │  (4-1 開發流程認證)                  \
  │                                      \
  ▼                                       ▼
CSA (4-2 組件安全)                  SSA (3-3 系統安全)
  │
  ▼
ICSA (CSA + IIoT 擴展)
```

> SDLA 是 CSA 和 SSA 的**前置條件**——產品供應商必須先證明開發流程合規（SDLA），再去認證產品（CSA）。但你可以同時送審兩者。

## 3. 認證流程

### 3.1 典型 CSA 認證流程

```
Step 1: 選擇認證機構 (ISO 17065 accredited)
        ├── exida (美國, 最知名)
        ├── TÜV Rheinland (德國)
        ├── TÜV SÜD (德國)
        └── CSSC (日本, Control System Security Center)

Step 2: 準備文件
        ├── SDLA 文件包（或現有的 SDLA 證書）
        ├── 產品安全需求規格（P2 SPR）
        ├── 威脅模型（P3 SD）
        ├── FSA-C (Functional Security Assessment) 自評表
        └── 測試報告（安全功能測試 + fuzz testing + pentest）

Step 3: 實驗室評估 (Lab Evaluation)
        ├── 文件審查（2-4 週）
        ├── 技術測試（由實驗室執行或監督）
        │   ├── 安全功能測試（照 FSA-C checklist）
        │   ├── 通訊穩定性測試（fuzz testing）
        │   └── 弱點掃描
        └── 報告撰寫

Step 4: 認證決定
        ├── 認證機構審查實驗室報告
        └── 發證（或要求補件）

Step 5: 維護
        ├── 證書有效期 3 年
        ├── 每年 surveillance audit
        └── 若產品有重大變更 → re-assessment
```

### 3.2 時間與費用概念

| 項目 | 估算範圍 | 備註 |
|---|---|---|
| SDLA 初次認證 | 3-6 個月, USD 30k-80k | 組織規模、成熟度影響大 |
| CSA 單一組件 | 2-6 個月, USD 20k-60k | 組件複雜度、FR coverage 影響大 |
| SSA 系統認證 | 4-9 個月, USD 50k-150k | 系統規模、整合複雜度 |
| 年度維護費 | ~USD 5k-20k/year | 含 surveillance audit |

> 以上為方向性估算，實際依認證機構報價為準。

## 4. 從哪開始？

### 4.1 建議路徑

```
階段 1: SDLA
  └── 投資開發流程改善（P1-P8）
  └── 拿到 SDLA 證書
  └── 耗時: 3-6 個月

階段 2: CSA（旗艦產品）
  └── 挑一個主力產品送 CSA
  └── 用 SDLA 證書滿足 4-1 部分（省重複審查）
  └── 耗時: 2-4 個月

階段 3: CSA（其他產品）或 SSA（系統）
  └── 產品線擴張認證
  └── 或走向系統級認證（SSA）
```

### 4.2 為什麼先做 SDLA？

- **一個 SDLA 證書受益全產品線**：拿到 SDLA 後，每新增一個產品的 CSA 認證不必重審開發流程
- **客戶要求 4-1 的頻率最高**：許多採購 RFP 寫「供應商開發流程需符合 IEC 62443-4-1」——直接引用 CCSC 4
- **內部改善優先**：沒有流程，產品認證會一直被退件（文件不全、測試不足）

## 5. 文件準備：最常被退件的點

| 退件原因 | 對應 Practice | 解法 |
|---|---|---|
| Security requirements not traceable to threats | P2 (SPR) | 建立 threat → requirement → test 可追溯矩陣 |
| Threat model too shallow | P3 (SD) | 必須完整 STRIDE 或同等方法，列出攻擊向量與對策 |
| No fuzz testing evidence | P5 (SVV) | 提供 fuzz testing 報告（工具、輸入、覆蓋率、結果） |
| Defect management process not documented | P6 (DM) | 拿出 defect tracking system 的流程文件 + 實例 |
| No hardening guide | P8 (SG) | 必須提供完整的 hardening guide |
| 自評 FSA-C 與實際測試結果矛盾 | CSA | 自評誠實為上——實驗室會複測 |

## 6. 小結

- ISASecure 是目前 IEC 62443 最具公信力的第三方認證
- SDLA → CSA → SSA 是三階段路徑，先拿 SDLA 省後續重複審查
- 時間 3-9 個月，費用 20k-150k USD（方案與產品複雜度決定）
- 最常被退件：文件與測試證據不足——不是產品沒做到，是拿不出證明

## 7. 下一篇

> [SL 達成路徑 — 從風險評估到合規驗證的完整閉環](02-sl-achievement-path.md)

產品拿到了 CSA。但對 Asset Owner 來說，**怎麼確保買回來的組件放進我的工廠後，真的達到 SL-T？**

---

相關：[CONTEXT.md](../../CONTEXT.md)、[ISASecure](https://www.isasecure.org/)
