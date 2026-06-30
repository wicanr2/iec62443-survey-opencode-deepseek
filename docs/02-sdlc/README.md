# 02 安全開發生命週期 (IEC 62443-4-1)

回答「產品怎麼安全地做出來」——從安全管理、需求規格、安全設計、安全實作、測試、漏洞處置、更新發布、到安全文件化。

## 閱讀順序

| # | 檔案 | 一句話 |
|---|---|---|
| 1 | [01-secure-sdlc-overview.md](01-secure-sdlc-overview.md) | SDLC 全景：8 Practices × 4 Maturity Levels |
| 2 | [02-security-management-requirements.md](02-security-management-requirements.md) | P1-2：安全管理（誰負責）+ 安全需求規格（從威脅導出） |
| 3 | [03-secure-design-implementation.md](03-secure-design-implementation.md) | P3-4：安全設計（STRIDE）+ 安全實作（編碼規範/SAST） |
| 4 | [04-security-testing-vnv.md](04-security-testing-vnv.md) | P5-6：安全測試（fuzz/pentest）+ 漏洞管理（CVE 流程） |
| 5 | [05-update-patch-management.md](05-update-patch-management.md) | P7-8：安全更新（簽章/反回滾）+ 安全文件化（hardening guide） |
| 6 | [06-maturity-levels.md](06-maturity-levels.md) | ML 1-4 深度解析 + 提升路徑 |


---

## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| **CVE** | Common Vulnerabilities and Exposures | 通用漏洞揭露編號 |
| **ML** | Maturity Level | 成熟度等級，IEC 62443-4-1 對開發流程的分級 (1-4) |
| **SAST** | Static Application Security Testing | 靜態應用安全測試，自動掃描原始碼找安全弱點 |
| **SDLC** | Secure Development Lifecycle | 安全開發生命週期，IEC 62443-4-1 規範 |
| **STRIDE** | Spoofing/Tampering/Repudiation/InfoDisclosure/DoS/Elevation | 微軟六面向威脅建模方法 |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
