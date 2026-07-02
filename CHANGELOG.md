# CHANGELOG

知識庫的版本演進。所有版本基於以下標準：

- **IEC 62443-4-2:2019**（Edition 1；ANSI/ISA 版為 2018）— Technical security requirements for IACS components。修訂案 `prAA:2026` 進行中。
- **IEC 62443-4-1:2018**（Edition 1）— Secure product development lifecycle requirements
- **ISASecure CSA 1.0.0**（2019-08-28）— Component Security Assurance certification scheme

> **雙編號提醒**：IEC 版（`IEC 62443-x-x`）與 ANSI/ISA 版（`ANSI/ISA-62443-x-x`）為平行編號，同一 Part 年份/型態可能不同；本庫以 IEC 版編號為主，年份/狀態以 [IEC Webstore](https://webstore.iec.ch) 最新公告為準。

## [0.2.0] — 2026-07-02

觸發：完整標準族「遺漏審查」。核心章原刻意聚焦 4-1/4-2；本版把視野擴到相鄰 Part 並修補既有內容的正確性缺口。

### 新增

- **Phase 6 超出核心**（`docs/06-beyond-4x/`，4 篇）
  - 01 系統安全要求 SR（IEC 62443-3-3:2013）— 完整 51 條 SR + RE/SL 階梯 + SSA
  - 02 風險評估 ZCR（IEC 62443-3-2:2020）— ZCR 七步 + ZCR5 十三子步 + SL-T 推導
  - 03 業主安全計畫（IEC 62443-2-1:2024）— SP/8 SPE + 成熟度模型 + 接 2-2
  - 04 IIoT 應用（IEC PAS 62443-1-6:2025）— PAS 定位 + 鬆動的假設 + ICSA
- **6 張新 SVG**（img/22–27）：ZCR 流程、SL-T 風險矩陣、SR/CR 兩層、RE 階梯、IACS vs IIoT、8 SPE 地圖
- `docs/README.md` 新增**範圍說明**（核心 vs 延伸，哪些 Part 刻意不涵蓋）

### 修正（遺漏審查第一級 — 正確性）

- **標準族全景表**補齊 1-2/1-4/1-6/2-2/6-1/6-2，加型態（IS/TR/TS/PAS）與 IEC/ISA 雙編號提醒（`01-overview` + `CONTEXT` + `img/01` 三處）
- 修正 **SL 定義錯置**：`-1-3` → `-1-1`
- ISASecure 篇補 **6-1:2024 / 6-2:2025 評估方法學** 與 **ACSSA（2024）** 認證方案
- 4-2 版本標註補 **IEC(2019)/ISA(2018) 雙編號 + prAA:2026** 修訂中
- **SPR 縮寫消歧義**：本庫自訂 SPR（P2）vs 官方 2-2 的 Security Protection Rating

### 已知限制（本版新增）

- 06 章多數斷言來自官方目次 / ISA 白皮書 / MDPI 論文交叉驗證；3-3 完整「51 SR × SL × RE」矩陣、2-1 各 SPE 逐條、1-6 條文級細節在付費正文內，未逐字核對，文中已標「待查證」。
- 版本狀態（2-2、6-x、1-6）以查證日 2026-07-02 的 IEC Webstore 公開資訊為準，可能變動。

## [0.1.0] — 2026-06-30

### 新增

- **Phase 1 基礎概念**（4 篇）
  - 01 IEC 62443 全景圖：為什麼 OT 需要獨立的資安標準
  - 02 Zone & Conduit：信任邊界的根本推導
  - 03 Security Levels：SL-T/SL-C/SL-A 三角閉環
  - 04 Foundational Requirements：FR 1-7 全景
- **Phase 2 SDLC**（6 篇）— IEC 62443-4-1 八大 Practice + ML 1-4 成熟度
- **Phase 3 組件 FR 逐條**（8 篇）— IEC 62443-4-2 四類組件 × 七條 FR
- **Phase 4 硬體專題**（3 篇）— Secure Boot、Tamper Protection、HSM/TPM/SE
- **Phase 5 合規認證**（4 篇）— ISASecure、SL 閉環、Gap 分析、標準對照
- **SL≥2 速查表**（1 篇）— FR 1-7 × SL 2-4 對照 + 10 項最低達標清單
- **Docker 部署安全 SL2 指南**（1 篇）— 11 項外部參考
- **GitHub 生態資源彙整**（1 篇）— 60+ 相關 repos

### 結構

- 5 個子目錄（01-foundations ~ 05-compliance）各加導航 README
- 21 張原創 SVG 架構圖
- 每篇 markdown 末加縮寫對照表

### 已知限制

- 核心 25 篇文章的外部引用集中在 Docker 文件，其他 24 篇以公開標準結構為主，未逐條對照原版標準條文。
- 25 篇 CR 條號（CR 1.1-7.8）來自 isecwire/iec62443-audit 與 VMware companion 交叉驗證，原版 62443-4-2 為付費標準，未逐條原文核對。
- 認證費用、實驗室選擇、ISASecure 認證機構列表等需以官方最新公告為準，本知識庫僅提供方向性估算。
- **已查證**（v0.1.1 新增）：CR 條號、SL 1-4 定義、CCSC 1-4 內容、IEC 62443-4-2:2019 發布日與 192 頁、IEC 62443-4-1:2018 發布日、ISASecure CSA 1.0.0 (2019-08-28) 均經 IEC Webstore 與 isasecure.org 官方確認。
- 詳見 [docs/REVIEW-reality-check.md](docs/REVIEW-reality-check.md) 完整真實性審查報告。

### 預計下版

- 0.2.0 — Round 2 質性 de-AI（91-deslop 三步 loop）+ 核心文章補外部引用
- 0.3.0 — Round 3 可行動 artifacts（Audit Checklist、Threat Model 範本、Hardening Guide 範本）
- 0.4.0 — Round 4 CI（GitHub Actions markdown-link-check + 表格一致性）+ CONTRIBUTING.md

[0.1.0]: https://github.com/wicanr2/iec62443-survey-opencode-deepseek/releases/tag/v0.1.0
