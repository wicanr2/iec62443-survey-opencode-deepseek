# IEC 62443 軟硬體安全規範知識庫

> 一句話定位：一份「從根本問題推導、圖文並茂、繁體中文」的 IEC 62443 知識庫——回答 **「軟硬體產品要拿到 IEC 62443 認證，我該做什麼、為什麼、東西要長成怎樣」**。
>
> 聚焦 **IEC 62443-4-2**（組件技術安全要求）與 **IEC 62443-4-1**（安全開發生命週期），輔以 -3-3（系統層）、-3-2（Zone & Conduit / SL-T）作為上下文。

## 從哪開始讀

| 出發點 | 讀這裡 | 為什麼 |
|---|---|---|
| 「IEC 62443 是什麼？為什麼我需要它？」 | [01-iec62443-overview.md](docs/01-foundations/01-iec62443-overview.md) | 從 IT/OT 的根本差異推導出為何工控需要獨立的資安標準 |
| 「Zone / Conduit / SL 怎麼設計？」 | [02-zone-and-conduit.md](docs/01-foundations/02-zone-and-conduit.md) → [03-security-levels.md](docs/01-foundations/03-security-levels.md) | 先建立信任邊界模型，再決定防到多強 |
| 「寫軟體的，我該注意什麼？」 | [02-sdlc/](docs/02-sdlc/) → [03-component-fr/](docs/03-component-fr/) | 先看開發流程要怎麼改（4-1），再看產品要滿足哪些技術要求（4-2） |
| 「做硬體的，Secure Boot / HSM / Tamper 怎麼做？」 | [04-hardware/](docs/04-hardware/) | 硬體專題：信任根、物理防護、安全元件 |
| 「要拿認證了，流程怎麼跑？」 | [05-compliance/](docs/05-compliance/) | ISASecure 認證體系、SL 達成路徑、gap 分析、標準對照 |
| 「一次搞清楚所有術語」| [CONTEXT.md](CONTEXT.md) | 術語表 (ubiquitous language) |

## 知識庫結構

```
iec62443-kb/
├── README.md              ← 你正在看：索引入口
├── PLAN.md                ← 建設計畫與進度
├── CONTEXT.md             ← 術語表
├── img/                   ← 所有 SVG 圖
└── docs/
    ├── 01-foundations/    ← 基礎概念（4 篇 ✅）
    │   ├── 01-iec62443-overview.md       全景圖 — 為什麼 OT 需要獨立的資安標準
    │   ├── 02-zone-and-conduit.md         Zone & Conduit — 信任邊界的根本推導
    │   ├── 03-security-levels.md          Security Levels — SL-T/SL-C/SL-A 三角
    │   └── 04-foundational-requirements.md FR 1-7 全景 — 七個基礎安全需求的由來
    ├── 02-sdlc/           ← IEC 62443-4-1 安全開發生命週期（6 篇 ✅）
    │   ├── 01-secure-sdlc-overview.md       SDLC 全景 + 認證角色
    │   ├── 02-security-management-requirements.md P1-2 安全管理 + 安全需求
    │   ├── 03-secure-design-implementation.md     P3-4 安全設計 + 安全實作
    │   ├── 04-security-testing-vnv.md             P5-6 安全測試 + 漏洞管理
    │   ├── 05-update-patch-management.md          P7-8 更新發布 + 安全文件化
    │   └── 06-maturity-levels.md                  ML 1-4 成熟度模型
    ├── 03-component-fr/   ← IEC 62443-4-2 組件 FR 逐條解說（8 篇 ✅）
    │   ├── 01-component-classification.md     組件分類與 CCSC 1-4
    │   ├── 02-fr1-identification-authentication.md FR1 識別與鑑別 (IAC)
    │   ├── 03-fr2-use-control.md               FR2 使用控制 (UC)
    │   ├── 04-fr3-system-integrity.md          FR3 系統完整性 (SI) + Secure Boot
    │   ├── 05-fr4-data-confidentiality.md      FR4 資料機密性 (DC) + 金鑰管理
    │   ├── 06-fr5-restricted-data-flow.md      FR5 限制資料流 (RDF)
    │   ├── 07-fr6-timely-response.md           FR6 事件回應 (TRE) + 稽核
    │   └── 08-fr7-resource-availability.md     FR7 資源可用性 (RA) + fail-safe
    ├── 04-hardware/       ← 硬體安全專題（3 篇 ✅）
    │   ├── 01-hardware-root-of-trust.md    硬體信任根與 Secure Boot 鏈
    │   ├── 02-physical-tamper-protection.md 物理防篡改與除錯埠管理
    │   └── 03-hsm-secure-element.md         HSM / TPM / Secure Element 選用指南
    └── 05-compliance/     ← 合規與認證實務（4 篇 ✅）
        ├── 01-isasecure-certification.md    ISASecure 認證體系
        ├── 02-sl-achievement-path.md        SL 達成路徑 — 閉環六步驟
        ├── 03-gap-analysis.md              常見合規落差與補強策略
        └── 04-cross-standard-mapping.md     與 ISO 27001 / NIST CSF / CC 對照
│
├── docs/iec62443-4-2-sl2-cheatsheet.md  ← IEC 62443-4-2 SL≥2 速查表：FR 1-7 × SL 2-4 對照 + SL 2 達標最低 10 項檢查表
│
└── (SmartMove-*.md)    ← [本地內部文件，不上傳 GitHub] SmartMove vs IEC 62443-4-2 SL≥2 差距分析
```

## 寫作原則

- **第一性原理**：每個概念先問「要解決什麼根本問題」，再推導解決方案
- **配 SVG**：流程 / 數學 / 對照一律白底手繪 SVG，橙點點綴，CJK 字型
- **繁體中文**：自然語言用繁體中文，標準條號 / 程式碼 / 識別符保留原文
- **不臆造**：不確定標「待查證」，不發明不存在的條號或工具名
- **柵欄原則**：發現「奇怪的」設計要求時，先重建它當初存在的必然性

## 素材來源

| 來源 | 用途 |
|---|---|
| IEC 官方 (webstore.iec.ch) | 標準結構、標題、發布年份 |
| ISASecure (isasecure.org) | 認證方案結構、組件分類定義、CCSC |
| Wikipedia (IEC 62443) | SL 定義、標準族全景摘要 |
| 實務經驗 (SmartMove / AMR / 工控部署) | 合規路面踩坑、gap 分析案例 |

> IEC 62443 標準全文為付費內容。本文庫以標準的公開結構、ISASecure 公開文件與業界已知概念為基礎撰寫，不複製標準原文。

---

**開始閱讀**：[第一篇 — IEC 62443 全景圖：為什麼 OT 需要獨立的資安標準](docs/01-foundations/01-iec62443-overview.md)
