# 04 硬體安全專題

回答「硬體層的安全基石怎麼做」——信任從哪裡開始、物理入侵怎麼防、金鑰存哪裡。

## 閱讀順序

| # | 檔案 | 一句話 |
|---|---|---|
| 1 | [01-hardware-root-of-trust.md](01-hardware-root-of-trust.md) | 硬體信任根：ROM→BL→OS→App 信任鏈、OTP/eFuse |
| 2 | [02-physical-tamper-protection.md](02-physical-tamper-protection.md) | 物理防篡改：機殼/網格/感測器 + JTAG 生命週期 |
| 3 | [03-hsm-secure-element.md](03-hsm-secure-element.md) | HSM / TPM / Secure Element 選用決策樹 + 實務整合 |


---

## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| **HSM** | Hardware Security Module | 硬體安全模組，專用加密金鑰管理硬體 |
| **JTAG** | Joint Test Action Group | 聯合測試行動組，晶片除錯介面標準 |
| **OS** | Operating System | 作業系統 |
| **OTP** | One-Time Programmable | 一次性可程式化記憶體 (eFuse 類) |
| **TPM** | Trusted Platform Module | 可信平台模組，平台身分與金鑰儲存 |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
