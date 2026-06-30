# CHANGELOG

知識庫的版本演進。所有版本基於以下標準：

- **IEC 62443-4-2:2019**（Edition 1）— Technical security requirements for IACS components
- **IEC 62443-4-1:2018**（Edition 1）— Secure product development lifecycle requirements
- **ISASecure CSA 1.0.0**（2019-08-28）— Component Security Assurance certification scheme

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
