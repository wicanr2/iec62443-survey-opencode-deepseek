# 03 組件 FR 逐條解說 (IEC 62443-4-2)

回答「軟硬體組件要滿足哪些技術安全要求」——以 7 條 FR 為骨架，每條展開軟體實作、硬體支援、SL 1-4 要求差異、常見不合格項目。

## 閱讀順序

| # | 檔案 | 一句話 |
|---|---|---|
| 0 | [01-component-classification.md](01-component-classification.md) | **先讀這篇**：四類組件（SA/ED/HD/ND）+ CCSC 1-4 通用約束 |
| 1 | [02-fr1-identification-authentication.md](02-fr1-identification-authentication.md) | FR1 (IAC)：你是誰？人/程序/裝置的鑑別 |
| 2 | [03-fr2-use-control.md](03-fr2-use-control.md) | FR2 (UC)：你能做什麼？RBAC + 最小權限 |
| 3 | [04-fr3-system-integrity.md](04-fr3-system-integrity.md) | FR3 (SI)：被偷改了嗎？Secure Boot + 簽章 |
| 4 | [05-fr4-data-confidentiality.md](05-fr4-data-confidentiality.md) | FR4 (DC)：被偷看了嗎？TLS + 金鑰管理 |
| 5 | [06-fr5-restricted-data-flow.md](06-fr5-restricted-data-flow.md) | FR5 (RDF)：誰能跟誰通？Default deny + 分區 |
| 6 | [07-fr6-timely-response.md](07-fr6-timely-response.md) | FR6 (TRE)：出事知道嗎？記錄 + 告警 + 稽核 |
| 7 | [08-fr7-resource-availability.md](08-fr7-resource-availability.md) | FR7 (RA)：還能運作嗎？抗 DoS + fail-safe |
