# FR 7 (RA) — 資源可用性

> 一句話定位：FR7 回答「系統還能繼續運作嗎」——抗 DoS、資源管理、故障安全 (fail-safe)、電源中斷恢復、冗餘備援。這是 OT 與 IT 安全最大的分野：在 IT，可用性是 SLA 問題；在 OT，失去可用性本身就是最嚴重的安全後果。
>
> 前置：[FR 6 (TRE)：事件回應與稽核](07-fr6-timely-response.md)

## 1. 根本問題

前面的 FR 1-6 假設「系統正常運作中，我們要防入侵」。但 FR 7 問的是：**如果系統被攻擊打到瀕死（或被自己 bug 搞掛），它能不能安全地降級、還是直接崩潰？**

OT 特有維度：

| 場景 | IT 後果 | OT 後果 |
|---|---|---|
| CPU 100% (DoS) | 網頁 500，客戶等一下 | **控制指令延遲或遺失 → 車不煞車 → 撞** |
| 記憶體耗盡 | process OOM kill，auto-restart | **控制邏輯中斷 → 閥門停在不明狀態** |
| 斷電 | UPS 撐到關機 | **必須能自動恢復到安全狀態，不需人工介入** |
| 安全功能本身失效 | 再登入一次就好 | **安全功能不能關閉**——即使系統瀕臨資源耗盡，安全控制不能降級 |

> FR7 的鐵律：**安全功能在資源耗盡時不能降級或關閉。** 這是它跟一般 IT 可用性（「盡量保持服務不中斷」）的根本差異。

## 2. SL-C 1-4 的要求差異

> 本 FR 對應 IEC 62443-4-2 中的 **8 條 CR**（CR 7.1 – CR 7.8）。每條 CR 對特定 SL-C 等級有要求，下表為各等級的綜合摘要。CR 條號與詳細要求請參閱標準原文或 ISASecure CSA-311。
>
> [¹]: 同上


| SL | DoS 防護 | 資源管理 | 故障安全 | 備援 |
|---|---|---|---|---|
| **SL-C 1** | — | — | 基本失效保護 | — |
| **SL-C 2** | 抗 moderate DoS | 基本資源監控 | fail-safe 設計 | — |
| **SL-C 3** | 抗 severe DoS | 資源使用上限/限制 | 驗證過的 fail-safe | 可支援備援 (standby) |
| **SL-C 4** | 抗 extreme DoS | 自動資源調節 | 自動安全復原 | Hot standby (無感切換) |

## 3. 軟體實作指南

### 3.1 抗 DoS：三個層級

| 層級 | 機制 | 說明 |
|---|---|---|
| **連線層** | 最大 concurrent connection 限制、SYN flood protection (SYN cookie) | 防止連線數耗盡 |
| **請求層** | Rate limiting per IP / per user、request queue 上限 | 防止 API 被打爆 |
| **協定層** | Modbus request rate limiting、工業協定 DPI 的 flood detection | 防止工業協定的 DoS |

### 3.2 資源管理：安全功能不能降級

```c
// 錯誤做法：資源快用完時關閉 auth 以維持服務
if (free_memory < CRITICAL_THRESHOLD) {
    disable_authentication();  // ❌ 這是後門
}

// 正確做法：拒絕新連線、保持現有的安全控制
if (free_memory < CRITICAL_THRESHOLD) {
    refuse_new_connections();   // ✓ 保護安全功能
    log_resource_exhaustion();  // ✓ 記錄事件 (FR6)
}
```

| 資源 | 管理策略 |
|---|---|
| **CPU** | 安全相關 task (auth, logging, watchdog) 用最高優先序；非安全 task 用 lower priority |
| **Memory** | 安全模組分配 fixed pool（不依賴 dynamic allocation）；OOM 時優先 kill 非安全 process |
| **Storage** | Log 用 circular buffer，避免 disk full；安全相關的 partition 要有保留空間 |

### 3.3 Fail-Safe：回到已知的安全狀態

故障安全狀態 = 當系統出錯時，設備必須回到**一個保證不會傷人的狀態**。

| 設備 | Fail-Safe 狀態 |
|---|---|
| AMR 控制器 | 失去通訊 → 立即減速至停車（不是繼續走最後一個指令） |
| PLC (製程) | CPU fault → 所有輸出 OFF（不是 hold last state） |
| 安全 PLC (SIS) | 任何異常 → 觸發安全停機 |
| 閥門 (fail-close) | 失去控制 → 閥門自動關閉（物理彈簧） |
| 網路設備 | 故障 → fail-open 或 fail-close？(需視設計，通常 fail-close 較安全) |

> **fail-safe ≠ fail-secure**：fail-safe 保護人（閥門關閉）；fail-secure 保護資產（門鎖上）。OT 優先 fail-safe。

### 3.4 電源中斷恢復

| SL | 要求 |
|---|---|
| SL-C 2+ | 斷電再上電後自動恢復到斷電前的安全狀態，不需人工介入 |
| SL-C 3+ | 恢復過程中做完整性檢查（firmware 簽章驗證、設定檔 checksum） |

## 4. 硬體支援需求

| 硬體功能 | 說明 |
|---|---|
| **Watchdog (internal + external)** | 雙重保護：internal watchdog 監控 CPU hang；external watchdog 監控整個系統（包括 power） |
| **Power fail detection** | 電壓下降時立即觸發 save state → 安全關機 |
| **Redundant power** | 雙電源模組，單一電源故障不影響運作 |
| **Hardware rate limiter** | 在 NIC/driver 層做硬體 rate limiting，不消耗應用 CPU |

## 5. 組件類型特化要點

| 類型 | RA 重點 |
|---|---|
| **SA** | 抗 API-level DoS；resource limit (goroutine / thread pool 上限)；graceful degradation |
| **ED** | 核心：watchdog + fail-safe 設計。資源極度受限時最難做：如何在 128KB RAM 中實現 fail-safe |
| **HD** | OS 層的 resource limit (cgroup, ulimit)；systemd watchdog；kernel panic → auto reboot |
| **ND** | 線速 forwarding 不因 DoS 衰減；control plane protection (CoPP) |

## 6. 常見不合格項目

1. **pprof / debug endpoint 可觸發遠端資源耗盡** — 無 rate limit 的 CPU profiling 就是 DoS 工具
2. **Watchdog 只監控 CPU hang，不監控應用層 deadlock** — 「CPU 沒當但邏輯卡死」是最難抓的
3. **無 fail-safe 設計** — 通訊中斷時設備「保持最後指令繼續跑」
4. **安全功能在資源用盡時降級** — 等同內建後門

## 7. 小結

- FR7 = 抗 DoS + 資源管理 + fail-safe + 電源恢復 + 備援
- 核心鐵律：安全功能在資源耗盡時**不能降級或關閉**
- fail-safe（保護人身安全）優先於 fail-secure（保護資產安全）
- Watchdog 必須雙層（internal + external），涵蓋 CPU hang 與應用層 deadlock

## 8. Phase 3 結束，下一篇

FR 1-7 逐條解說完成。下一篇進入 **Phase 4：硬體安全專題** → [硬體信任根與 Secure Boot 鏈](../04-hardware/01-hardware-root-of-trust.md)（撰寫中）

---

相關：[CONTEXT.md](../../CONTEXT.md)、[IEC 62443-4-2 官方頁](https://webstore.iec.ch/en/publication/34421)


---

## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| **AMR** | Autonomous Mobile Robot | 自主移動機器人/搬運車 |
| **CR** | Component Requirement | 組件安全需求，IEC 62443-4-2 定義 |
| **CSA** | Component Security Assurance | ISASecure 組件安全認證 |
| **DPI** | Deep Packet Inspection | 深層封包檢測，辨識應用層協定內容 |
| **DoS** | Denial of Service | 服務阻斷攻擊，耗盡系統資源使其無法回應 |
| **ED** | Embedded Device | 嵌入式裝置組件 (IEC 62443-4-2 組件類型) |
| **FR** | Foundational Requirement | 基礎安全需求，IEC 62443 的核心架構，共 7 條 (FR1-7) |
| **HD** | Host Device | 主機裝置組件 (IEC 62443-4-2 組件類型) |
| **ISASecure** | ISA Security Compliance Institute | ISA 資安合規協會，營運 IEC 62443 認證方案 |
| **ND** | Network Device | 網路裝置組件 (IEC 62443-4-2 組件類型) |
| **NIC** | Network Interface Card | 網路介面卡 |
| **OS** | Operating System | 作業系統 |
| **PLC** | Programmable Logic Controller | 可程式邏輯控制器 |
| **RA** | Resource Availability | 資源可用性 (FR7) |
| **SA** | Software Application | 軟體應用組件 (IEC 62443-4-2 組件類型) |
| **SL** | Security Level | 安全等級，依攻擊者能力分 0-4 級 |
| **SL-C** | Capability Security Level | 能力安全等級，組件或系統能達到的安全等級 |
| **TRE** | Timely Response to Events | 事件及時回應 (FR6) |

> 完整術語表見 [CONTEXT.md](../../CONTEXT.md)
