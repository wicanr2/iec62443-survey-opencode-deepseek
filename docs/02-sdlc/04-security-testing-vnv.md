# Practice 5-6：安全測試與漏洞管理

> 一句話定位：P5 (SVV) 驗證安全功能真的有效、找系統性漏洞；P6 (DM) 處理「漏洞發現後到修補發布」的完整流程。一條做測試、一條處置結果——兩者構成安全品質的閉環。
>
> 前置：[Practice 3-4：安全設計與安全實作](03-secure-design-implementation.md)
> 下一篇：[Practice 7-8：更新發布與安全文件化](05-update-patch-management.md)

## 1. P5 (SVV) — 安全驗證與測試 (Security V&V)

### 1.1 根本問題

「安全功能有寫、但沒測」等於沒寫。更糟的是：code 裡有安全漏洞、但你沒測過所以不知道、客戶先發現。

P5 要求對安全功能做**結構化的測試**——不只是 QA 手點幾下，而是系統性涵蓋 functional、non-functional、adversarial 三個面向。

### 1.2 Practice 5 的測試層級

| 測試類型 | 層級 | 說明 | 工控特化 |
|---|---|---|---|
| **安全功能測試** | 功能性 | 驗證每個安全功能（auth、ACL、加密、稽核）照規格運作 | 裝置憑證註冊流程、RBAC role 測試、TLS handshake 驗證 |
| **負面測試** | 功能性 | 驗證錯誤輸入能被正確拒絕（輸入驗證、邊界測試） | 畸形的 Modbus/DNP3 封包、超大 payload |
| **Fuzz Testing** | 對抗性 | 自動化產生大量無效/隨機輸入，看系統是否崩潰 | **工業協定 fuzzing**（Modbus、EtherNet/IP、OPC UA）是特殊領域 |
| **滲透測試** | 對抗性 | 模擬真實攻擊者嘗試突破防禦 | 從企業網路橫向移動到控制網路、物理入侵攻擊 |
| **弱點掃描** | 對抗性 | 用自動化工具掃描已知 CVE | 掃描 OS、library、通訊協定 stack 的已知漏洞 |
| **回歸測試** | 維護性 | 每次變更後確保安全功能沒被改壞 | — |

### 1.3 工控 fuzz testing 的特殊性

IT 世界的 fuzz testing 通常針對 HTTP、JSON parser、XML parser 等。工控領域需要針對**工業通訊協定**做 fuzz：

| 協定 | Fuzz 目標 | 工具 |
|---|---|---|
| Modbus TCP | Function code、register address range、length field overflow | Boofuzz、自訂 Sulley fuzzer |
| DNP3 | 每個 DNP3 object header 欄位、sequence number rollover | AFL + DNP3 harness |
| EtherNet/IP | CIP connection packet 的畸形參數 | 自訂 scapy-based fuzzer |
| OPC UA | 安全通道建立過程中的憑證格式錯誤 | OPC UA fuzzer (OPC Foundation 提供) |
| CAN bus | 惡意 CAN frame ID、DLC 長度不一致 | CANard、ICSIM |

> **為什麼 CTF-style pentest 不夠？** 滲透測試是一個人、在某個時間點、用他知道的手法測試。Fuzz testing 是百萬個 random input 轟炸。兩者互補——pentest 找邏輯漏洞，fuzz 找記憶體崩潰與 parser 漏洞。

### 1.4 安全測試文件化

P5 不只是「跑測試」，還需要產出文件來證明：
- 測試計畫（test plan）：哪些安全功能要測、用什麼方法、什麼工具
- 測試報告：結果（pass/fail）、如果 fail 的原因、對應的 defect ID
- 可追溯矩陣：安全需求（P2）→ 設計決策（P3）→ 測試項目（P5）

> 這對 ISASecure SDLA/CSA 認證審查非常重要——審查員要看「你有沒有測到你宣稱的安全功能」。

## 2. P6 (DM) — 缺陷管理 (Defect Management)

### 2.1 根本問題

漏洞發現了。然後呢？

工控的兩難：
- 修太快：patch 沒測夠→上線後搞掛產線→誰負責？
- 修太慢：已知漏洞擺著一個月→客戶被入侵→誰負責？

P6 定義了一個結構化的「漏洞→修復→發布」流程，讓組織有明確的 SLA 和可重複的步驟，而不是靠個人判斷。

### 2.2 Practice 6 的核心流程

```<p align="center"><img src="../../img/07-defect-management-flow.svg" width="800" alt="P6 漏洞處理流程 (Defect Management)"></p>```

### 2.3 Severity × 回應時限

| Severity | 範例 | 回應時限（建議） | 對應 SL |
|---|---|---|---|
| **Critical** | RCE、未認證即可觸發、影響安全功能 | ≤ 7 天出 patch | SL 3-4 產品 |
| **High** | DoS、繞過認證（需先登入）、資訊洩漏 | ≤ 14 天 | SL 2-3 |
| **Medium** | 需特定條件才能觸發、風險較低 | ≤ 30 天 | SL 2 |
| **Low** | 無實際攻擊路徑、理論性 | 排進下一個 release | 全部 |

> 時限為建議值，非 IEC 標準強制。實務上合約可能要求更嚴格的 SLA。

### 2.4 工控環境的 DM 特殊挑戰

| 挑戰 | 說明 | 解法 |
|---|---|---|
| **客戶不更新** | 產線不能停，客戶可能拖半年才排 patch | 在 product advisory 中明確標風險、提供 mitigation（如關閉受影響服務）直到 patch 完成 |
| **Legacy 元件** | 產品用了 10 年前的 codebase，已經沒人懂那段 code | 強制 code ownership 繼承、文件化關鍵模組 |
| **第三方 lib 的 CVE** | 你的產品依賴的 open source library 有 CVE，但 library 維護者不修 | fork + 自行修復、或替換 library |
| **Coordinated Disclosure** | 外部研究員回報漏洞但不給公開時間限期 | 建立正式的漏洞通報政策與 PGP key 放在官網 |

## 3. 小結

- **P5 (SVV)**：安全功能測試 + 對抗性測試 (fuzz/pentest) + 弱點掃描——保證「說有做的安全功能真的有效」
- **P6 (DM)**：漏洞→Triage→修→回歸→發布——保證「發現漏洞後能有序修補」
- 兩者與 P7 接續：P6 修完的 patch，由 P7 (SUM) 安全地發布給客戶

## 4. 下一篇

> [Practice 7-8：更新發布與安全文件化](05-update-patch-management.md)（撰寫中）

漏洞修完了，patch 怎麼安全地送到客戶手上、又不被中途篡改？

---

相關：[CONTEXT.md](../../CONTEXT.md)、[IEC 62443-4-1 官方頁](https://webstore.iec.ch/en/publication/33615)
