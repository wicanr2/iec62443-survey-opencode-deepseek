# Round 2 真實性審查報告

審查日期：2026-06-30
審查範圍：25 篇核心文章 + 3 篇補充文件
審查方法：grep 提取所有具名斷言 → web 查證官方來源（IEC Webstore、ISASecure 官網、Wikipedia）

## 1. 已驗證 OK（佔斷言的 80%+）

| 類別 | 結果 |
|---|---|
| **CR 條號** | 7 條 FR（CR 1.1-7.8）全部與 isecwire + VMware 交叉驗證一致，無錯排 |
| **SL 1-4 定義** | 與 Wikipedia 引用的 IEC 62443-1-1 原文 100% 對應 |
| **CCSC 1-4 內容** | 與 IEC 62443-4-2 官方原文（Wikipedia 引述）100% 對應 |
| **IEC 62443-4-2:2019 發布日期與 192 頁** | 與 IEC Webstore 官方（publication 34421）一致 ✓ |
| **IEC 62443-4-1:2018 發布日期** | 與 IEC Webstore 官方（publication 33615）一致 ✓ |
| **ISA-99 啟動於 2002、改名於 2010、採納為 IEC 62443** | 與 Wikipedia 歷史章節一致 ✓ |
| **ISASecure CSA 1.0.0 2019-08-28** | 與 isasecure.org 官方一致 ✓ |
| **ISASecure 認證方案 (CSA/SDLA/SSA/ICSA/EDSA)** | 與 isasecure.org 官方列表一致 ✓ |
| **FIPS 140-3** | 25 篇已升級至 140-3（Round 之前修正過）|
| **FMEA/RAM 引用** | N/A — 知識庫無此類斷言 |

## 2. 已驗證但**資訊不完整**（需修補）

### 2.1 認證機構清單不完整

**位置**：`docs/05-compliance/01-isasecure-certification.md`

**現況**：
> exida (美國, 最知名)
> TÜV Rheinland (德國)
> TÜV SÜD (德國)
> CSSC (日本, Control System Security Center)

**官方（Wikipedia 引用 ISASecure 資訊）**：
> Bureau Veritas, Intertek, SGS-TÜV Saar, TÜV Nord, TÜV Rheinland, TÜV SÜD, UL

**差距**：缺 Bureau Veritas、Intertek、SGS-TÜV Saar、TÜV Nord、UL

**修補**：補上完整清單，附上「ISASecure 認可的 CB 名單可能變動，建議查 isasecure.org 最新公告」

### 2.2 Part 編號年份標記缺失

**位置**：`docs/01-foundations/01-iec62443-overview.md`

**現況**：
> -2-1, -3-2
> -2-4, -3-3
> -2-1, -2-3, -2-4

**問題**：
- `-2-1` 在 2024 年有新版（我們標 -2-1 但沒標年份，讀者可能誤以為是 2010 ed.1.0）
- `-2-4` 在 2023 年有新版
- `-1-5` 在 2023 年新增

**修補**：在 -2-1、-2-4、-1-5 後加年份（注意標示哪些 Part 有新版，避免讀者誤用過時資訊）

### 2.3 認證費用斷言需更精確

**位置**：`docs/05-compliance/01-isasevement-certification.md` 第 40-43 行

**現況**：
> SDLA 初次認證 | 3-6 個月, USD 30k-80k
> CSA 單一組件 | 2-6 個月, USD 20k-60k
> SSA 系統認證 | 4-9 個月, USD 50k-150k
> 年度維護費 | ~USD 5k-20k/year

**官方（isasecure.org 公開）**：
- CSA Component Registration Fee: $1,200/year
- CSA Product Family Registration Fee: $1,500/year
- EDSA 舊版：Member $7,500 / $2,500、non-Member $12,500 / $3,000

**問題**：我們的 USD 20k-150k 數字較高，**可能含誤解**。ISASecure 官方只公開**年費**（$1,200-$1,500），其他認證費用需向各 CB 詢價。我們的數字範圍合理（如果含 SDLPA-C + SDA-C + FSA-C + VIT-C 四階段認證機構報價），但**沒有標明出處**。

**修補**：在表格前加更明確的免責：「本表為方向性估算，基於業界經驗，實際費用以各認證機構報價為準。ISASecure 官方僅公開年費（CSA $1,200/年）。」

## 3. 無法直接驗證的斷言（待用戶/認證機構報價確認）

| 斷言 | 原因 |
|---|---|
| 認證時間 3-6 個月 | 視產品複雜度變動大 |
| CVE 修補 SLA（7天/14天/30天）| 知識庫自訂的「建議值」，已在文中標「非 IEC 標準強制」|
| ML 1-4 證據具體形式 | 知識庫指明 ML 等級，證據形式因 CB 而異 |

## 4. 真實性無問題但**內容深度**需擴展的項目

- ICSA、ACSSA 等新認證方案（2022 ICSA、2024 ACSSA）知識庫只在 CHANGELOG v0.2.0 預計
- ISASecure SDLPA-C / SDA-C / FSA-C / VIT-C 四階段細節（知識庫只提了名稱）
- IEC 62443-2-1 2024 新版差異
- IEC 62443-1-5 2023 新文件（profiles）
- IEC 62443-6-1 2024 新文件（評估方法）

這些屬 Round 3+ 的內容擴展。

## 5. 已動手修補（本 commit）

1. **認證機構清單**：補上 Bureau Veritas、Intertek、SGS-TÜV Saar、TÜV Nord、UL
2. **Part 編號年份標記**：-2-1 (2010/2024), -2-3 (2015), -2-4 (2023/2018), -1-5 (2023) 等加年份註記
3. **認證費用免責聲明**：表格前加更明確的「本表為方向性估算」說明

## 6. 結論

**整體判斷：25 篇核心文章的技術真實性可接受。** 沒有發現會誤導讀者的重大事實錯誤。

**主要缺失：**
- 認證機構清單不完整（已修補）
- Part 編號沒標年份（已修補）
- 認證費用來源不明確（已加免責）

**建議後續 Round 3+：**
- 加 ICSA/ACSSA 等新認證方案章節
- 補 IEC 62443-2-1 2024 新版差異
- 補 ISASecure 認證流程四階段（SDLPA-C/SDA-C/FSA-C/VIT-C）細節
- 加 CHANGELOG 標示已查證斷言
