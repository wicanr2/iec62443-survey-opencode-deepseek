# 🔐 IEC 62443 — 工控安全，從第一性原理開始

<p align="center">
  <b>25 篇深度文章 · 20 張原創 SVG 架構圖 · 3,600+ 行繁體中文</b><br>
  <sub>覆蓋 IEC 62443-4-2（組件安全）× IEC 62443-4-1（安全開發）× 硬體信任根</sub>
</p>

---

> **你不是在背條文。你是在重建每一個安全要求存在的必然性。**

IEC 62443 是工控資安的「聖經」——但原文 192 頁英文、付費、滿滿條號，對軟硬體工程師極不友善。

這份知識庫做三件事：
1. **從「為什麼」開始**：每個概念先問「它要解決什麼根本問題」，再推導答案——不是給你一張 checklist 叫你去填
2. **軟硬體雙軌**：FR 1-7 每條都拆成「軟體怎麼實作」和「硬體要支援什麼」，附具體的 code/config/hardware 決策
3. **從概念到認證一條線**：Zone & Conduit → SL-T/C/A 閉環 → Secure Boot 信任鏈 → ISASecure 送審文件包

---

## 🧭 你從哪條路進來？

| 你的身份 | 直接去這裡 | 5 秒說明 |
|---|---|---|
| 🆕 第一次接觸 IEC 62443 | **[01 全景圖 →](docs/01-foundations/01-iec62443-overview.md)** | IT vs OT 的根本差異，Stuxnet 為什麼是轉捩點 |
| 🏗️ 系統架構師 | **[02 Zone & Conduit →](docs/01-foundations/02-zone-and-conduit.md)** | 工廠網路怎麼切分區、怎麼設防火牆規則 |
| 🔑 CTO/資安長 | **[SL 速查表 →](docs/iec62443-4-2-sl2-cheatsheet.md)** | 10 分鐘看懂 SL 2-4 所有要求 + 10 項最低達標清單 |
| 💻 後端/全端工程師 | **[03 FR 逐條 →](docs/03-component-fr/README.md)** | FR1 認證、FR4 TLS、FR5 CORS——每條有 code 範例和反模式 |
| 🔩 嵌入式/硬體工程師 | **[04 硬體專題 →](docs/04-hardware/README.md)** | Secure Boot、JTAG 鎖、HSM vs TPM vs SE 選型 |
| 📋 專案經理/合規 | **[05 認證實務 →](docs/05-compliance/README.md)** | ISASecure CSA/SDLA 怎麼跑、多少錢、文件怎麼準備 |
| 🌐 找開源工具 / 生態資源 | **[GitHub 資源彙整 →](docs/github-iec62443-resources.md)** | 60+ repos：合規檢查表、審計腳本、Lab 環境、Secure Boot 實作 |

---

## 📐 知識庫結構

```
📦 iec62443-kb/                       25 篇文章 · 20 張 SVG
│
├── 📂 docs/                          文件入口（→ docs/README.md）
├── 🧱 01-foundations/   (4 篇)       OT 全景 → Zone&Conduit → SL → FR1-7
├── 🔄 02-sdlc/          (6 篇)       4-1 八大 Practice + ML 1-4 成熟度
├── 🎯 03-component-fr/  (8 篇)       4-2 四類組件 × 七條 FR 逐條
├── 🔩 04-hardware/      (3 篇)       Secure Boot · Tamper · HSM/TPM/SE
├── 📜 05-compliance/    (4 篇)       認證流程 · SL 閉環 · Gap 分析 · 標準對照
│
├── 📊 iec62443-4-2-sl2-cheatsheet.md    FR 1-7 × SL 2-4 速查 + 10 項達標清單
├── 🌐 github-iec62443-resources.md       GitHub 生態資源彙整 (60+ repos)
├── 🐳 docker-iec62443-sl2-hardening.md    Docker 部署安全：如何達到 SL2
├── 📖 CONTEXT.md                        完整術語表
└── 🗺️ PLAN.md                           建設計畫（已 100% 完成）
```

---

## ⚡ 為什麼這份知識庫跟別人不一樣

| 一般 IEC 62443 資源 | 這份知識庫 |
|---|---|
| 條文翻譯、bulleted list | **第一性原理推導**——先問「為什麼是這個設計」 |
| 純文字、難以消化 | **20 張手繪 SVG**——流程圖、狀態機、信任鏈全部可視化 |
| IT 通用、不分工控場景 | **工控特化**——每條 FR 都有 Modbus fuzz、PLC JTAG lock、AMR fail-safe 實例 |
| 概念層、無法落地 | **具體到 code/config/hardware 層級** |
| 全英文 | **繁體中文**行文，術語首次出現即翻譯 |

---

## 📋 寫作紀律

- **第一性原理**：先問根本問題，再推導解答——不堆事實
- **SVG 配圖**：流程/架構/狀態機一律白底手繪，橙點點綴
- **不臆造**：不確定的標「待查證」，不發明不存在的條號
- **柵欄原則**：碰到奇怪的設計要求時，先重建它存在的必然性
- **CR 條號可追溯**：7 篇 FR 皆對照 IEC 62443-4-2 CR 編號（來源交叉驗證）

---

## 🔗 素材來源

| 來源 | 用途 |
|---|---|
| [IEC 官方](https://webstore.iec.ch) | 標準結構、標題、發布年份 |
| [ISASecure](https://www.isasecure.org) | 認證方案、組件分類定義、CCSC |
| [isecwire/iec62443-audit](https://github.com/isecwire/iec62443-audit) | CR 條號交叉驗證 |
| Wikipedia (IEC 62443) | SL 定義、標準族摘要 |

> IEC 62443 標準全文為付費內容。本文庫以標準的公開結構、ISASecure 公開文件與業界已知概念為基礎撰寫，不複製標準原文。

---

<p align="center">
  <a href="docs/01-foundations/01-iec62443-overview.md"><b>🚀 開始閱讀：第一篇 — 為什麼 OT 需要獨立的資安標準</b></a>
  <br><sub>或者直接跳去 <a href="docs/iec62443-4-2-sl2-cheatsheet.md">SL≥2 速查表</a></sub>
</p>
