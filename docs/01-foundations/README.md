# 01 基礎概念

在進入軟硬體的技術要求（4-1、4-2）之前，你需要先理解三件事：
1. 為什麼 OT（工控）需要自己的一套資安標準，不能直接套 IT 的？
2. 在一個工廠裡，信任邊界怎麼劃？（Zone & Conduit）
3. 「夠安全」是多安全？怎麼量化？（Security Levels）
4. 安全的面向有哪些？怎麼分解？（FR 1-7）

## 閱讀順序

| # | 檔案 | 一句話 |
|---|---|---|
| 1 | [01-iec62443-overview.md](01-iec62443-overview.md) | 全景圖：IT vs OT、Stuxnet 轉捩點、IEC 62443 標準族地圖 |
| 2 | [02-zone-and-conduit.md](02-zone-and-conduit.md) | 信任邊界：為什麼不能一張平面網路打天下 |
| 3 | [03-security-levels.md](03-security-levels.md) | 安全等級：SL-T/SL-C/SL-A 三層閉環 + SL 1-4 威脅模型 |
| 4 | [04-foundational-requirements.md](04-foundational-requirements.md) | FR 1-7：七個基礎安全需求的由來與全景矩陣 |

→ 回 [README](../../README.md)


---

## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-A** | Achieved Security Level | 達成安全等級，部署後經評估確認的實際等級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| **SL-T** | Target Security Level | 目標安全等級，業主經風險評估後設定 |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
