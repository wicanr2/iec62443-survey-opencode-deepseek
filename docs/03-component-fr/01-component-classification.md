# 組件分類全景 — 四類組件的邊界與 CCSC

> 一句話定位：IEC 62443-4-2 不把「軟體」和「硬體」當成同一種東西來要求——它先分類（SA/ED/HD/ND），再對每類給不同的 CR 適用性。本篇定義四類組件的邊界與四個通用約束 (CCSC 1-4)。
>
> 前置：[FR 1-7 全景](../01-foundations/04-foundational-requirements.md)
> 下一篇：[FR 1 (IAC)：識別與鑑別控制](02-fr1-identification-authentication.md)

## 1. 根本問題：為什麼要分類？

一台跑 Linux 的工業電腦（Host Device）和一台跑 bare-metal firmware 的 PLC（Embedded Device）的安全要求不會一樣。前者有 OS 級的防護（user permissions、SELinux、firewall daemon）；後者只有自己寫的那幾 KB code——所有安全功能都要手動實作。

不分類的後果：
- 對 firmware MCU 要求「跑 antivirus」→ 不可能（沒有 OS、沒有 filesystem、沒有 CPU 資源）
- 對 pure software application 要求「secure boot」→ 不合理（它不控制開機流程，那是 host OS 的事）

**分類的目的不是貼標籤，而是精準地只要求「這個類別能做得到的事」。**

## 2. 四類組件定義

| 類型 | 縮寫 | 定義 | 關鍵特徵 |
|---|---|---|---|
| **Software Application** | SA | 一個或多個軟體程式及其相依項，用於與製程或控制系統互動 | 沒有自己控制的硬體；跑在 host OS 上；安全倚賴底層 OS 與硬體 |
| **Embedded Device** | ED | 執行嵌入式軟體的特殊用途裝置，直接監控/控制/致動工業製程 | 專用硬體；firmware 是唯一軟體；所有安全功能要自己實作；常無 OS 或只有 RTOS |
| **Host Device** | HD | 執行通用 OS 的裝置，可承載軟體應用、資料儲存或功能 | 跑 Linux/Windows；可裝軟體/防毒；OS 提供部分安全功能 |
| **Network Device** | ND | 促進裝置間資料流或限制資料流的裝置 | 不直接控制製程；核心功能是轉送/過濾封包；embedded OS |

### 2.1 邊界案例：怎麼區分 ED vs HD？

| 情境 | 分類 | 原因 |
|---|---|---|
| 跑 Linux 的 Raspberry Pi 控制一個閥門 | **HD** | 有通用 OS、可遠端登入、可裝軟體 |
| STM32 bare-metal firmware 控制一個閥門 | **ED** | 無 OS、專用 firmware |
| 一台工業 HMI（Windows Embedded + 觸控介面） | **HD** | 有 OS、可裝軟體 |
| 一台 Modbus TCP 轉 RTU 的 gateway（firmware） | **ND** | 核心功能是轉送封包 |

### 2.2 邊界案例：SA 還是 ED？

| 情境 | 分類 | 原因 |
|---|---|---|
| SCADA 軟體（安裝在 Windows 上） | **SA** | 純軟體，倚賴 OS |
| 一個 PLC 的 web-based 管理後台（內建在 firmware 中） | 歸入 **ED** | 管理後台是 ED 的一部分，跟 ED 一起認證 |

> 關鍵原則是：**一個產品可能同時是多個類型**（例如一台 HMI = HD + 內建 SA）。認證時會對每個面向做評估。

## 3. CCSC 1-4：四個通用約束

CCSC（Common Component Security Constraints）是四條對**所有組件類型**都適用的鐵律。把這些放在第一篇，因為後面每個 FR 的解說都會回來引用它們。

### CCSC 1：考量系統安全特性

> 組件須考量其所在系統的一般安全特性。

**翻譯**：你開發一台 PLC，你的 threat model 不能假設「這台 PLC 放在一個完全安全、沒人會攻擊的環境」。你要假設它被部署在一個真實的工廠裡——有可能內網已被入侵、有可能隔壁的 PC 中了 malware。

**實務影響**：設計 threat model 時，要考慮組件「部署後的環境」。例如預設的網路服務應該全部關閉（default deny），因為你不知道客戶會把它放在哪個 subnet。

### CCSC 2：系統層補償

> 組件無法自行滿足的技術需求，可由系統層級補償對策滿足——但須記載於組件文件中。

這是最重要、也最常被誤解的一條。見 [Zone & Conduit 篇](../01-foundations/02-zone-and-conduit.md) 的詳細說明。

**實務影響**：如果你的產品不支援某個 CR（例如不支援 TLS），你必須在文件（P8 SG hardening guide）中寫明「本組件不支援傳輸加密，請在 Conduit 上透過 VPN/IPsec 補償」。

### CCSC 3：最小權限原則

> 組件須套用最小權限原則 (Least Privilege)。

每個程序、每個服務、每個使用者只拿到完成其任務所需的**最少權限**。

**實務影響**：
- 預設不要用 root/admin 執行服務
- 通訊埠只開必要的，關掉 debug port
- API endpoint 只暴露需要的，不要全部 CRUD 都開
- 組件出廠時「所有功能關閉、逐項開啟」而非「全開、逐項關閉」

### CCSC 4：安全開發流程

> 組件須由符合 IEC 62443-4-1 的開發流程開發與支援。

這條的詳細內容就是整個 [Phase 2 SDLC 系列](../02-sdlc/01-secure-sdlc-overview.md)。

**實務影響**：對軟硬體開發者的核心約束。你的產品取得 CSA 認證前，你的開發流程必須先過 4-1。

## 4. 組件類型 × FR 適用性

不是每條 FR 對每類組件的要求都一樣。同一個 FR 1 (IAC)：
- 對 **SA**：主要管「人類使用者登入」和「API 之間的認證」
- 對 **ED**：除了上面，還管「裝置本身的鑑別」（這台 PLC 真的是工廠裡那台，不是被換過的）
- 對 **ND**：管「管理介面的認證」和「裝置之間的認證」

後續每篇 FR 會標明**各組件類型的特化要求**。

## 5. 下一篇

> [FR 1 (IAC)：識別與鑑別控制](02-fr1-identification-authentication.md)

第一個 FR：系統必須知道「你是誰」——人、程式、裝置，三種主體各有不同的鑑別方法。軟硬體各要支援什麼？

---

相關：[CONTEXT.md](../../CONTEXT.md)、[ISASecure CSA 認證](https://www.isasecure.org/en-US/Certification/IEC-62443-CSA-Certification)
