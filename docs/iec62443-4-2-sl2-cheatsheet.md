# IEC 62443-4-2 SL ≥ 2 重點整理

> 一句話：SL 2 是 IEC 62443-4-2 的分水嶺——SL 1 只防意外，SL 2 起要求防**蓄意攻擊**。本篇將七條 FR 在 SL 2/3/4 的核心要求濃縮成一張可查閱的對照表，附每條的常見不合格項。

---

## 1. SL 分水嶺：為什麼 SL 2 是起跑線

| SL | 防什麼 | 攻擊者 | 4-2 要求的本質變化 |
|---|---|---|---|
| SL 1 | 意外/誤觸 | 無蓄意 | 基本識別、無須強鑑別 |
| **SL 2** | **一般駭客** | 蓄意、簡單手段、低資源 | **強制鑑別 + 簽章 + 加密 + 分區** |
| SL 3 | 針對性攻擊 | 專精 OT、中資源 | MFA + Secure Boot + 完整金鑰管理 |
| SL 4 | 國家級 | 大量資源、APTs | 硬體信任根 + 雙人授權 + 抗量子準備 |

> 實務上，半導體/關鍵基礎設施的採購 RFP 最低要求就是 SL 2-3。SL 1 對這類客戶沒有意義。

---

## 2. FR 1-7 × SL 2-4 需求速查表

### FR1 (IAC) — 識別與鑑別

| 要求 | SL 2 | SL 3 | SL 4 |
|---|---|---|---|
| **使用者鑑別** | 帳密強制 + 複雜度規則 + 帳號鎖定 | SL2 + **MFA** + 防 replay | SL3 + 硬體 token (FIDO2/smart card) |
| **裝置鑑別** | PSK 或簡易憑證 | **x509 mTLS** 雙向 + 憑證生命週期管理 | SL3 + **硬體安全模組 (SE/HSM)** 保護私鑰 |
| **預設密碼** | 出廠隨機密碼或首次開機強制修改 | 同左 | 同左 |
| **程序鑑別** | API key / service token | OAuth client credential + mTLS | 硬體綁定 token |

**常見不合格**：預設 admin/admin、JWT secret hardcode、MAC address 當裝置認證

---

### FR2 (UC) — 使用控制

| 要求 | SL 2 | SL 3 | SL 4 |
|---|---|---|---|
| **授權模型** | RBAC（多角色）| SL2 + **職責分離 (SoD)** | SL3 + 動態授權 + **雙人授權** |
| **最小權限** | 強制 | 強制 + 稽核證明 | 強制 + 自動檢查 |
| **Session 管理** | 閒置逾時自動鎖定 | SL2 + 最大 concurrent session 限制 | SL3 + 遠端強制登出 |

**常見不合格**：DevMode flag 全繞過、所有人 admin、無 session timeout

---

### FR3 (SI) — 系統完整性

| 要求 | SL 2 | SL 3 | SL 4 |
|---|---|---|---|
| **韌體/軟體簽章** | 數位簽章驗證（非 CRC）| SL2 + **anti-rollback** | SL3 + runtime integrity check |
| **Secure Boot** | 建議（非強制）| **強制**（信任鏈驗證每一層）| SL3 + **硬體信任根** (OTP/eFuse) |
| **輸入驗證** | 型別/長度驗證 | SL2 + 白名單驗證 | SL3 + 自動輸入過濾 |
| **惡意程式防護** | — | Malware detection | Malware prevention |
| **JTAG/SWD** | 量產後禁用（fuse lock）| 同左 | 永久禁用 |

**常見不合格**：FW 更新無簽章、JTAG 未鎖、CRC 當簽章用、無 anti-rollback

---

### FR4 (DC) — 資料機密性

| 要求 | SL 2 | SL 3 | SL 4 |
|---|---|---|---|
| **傳輸加密** | **TLS 1.2+**（禁用 SSLv3/TLS1.0/1.1/RC4/3DES）| **TLS 1.3** or IPsec | SL3 + 抗量子準備 |
| **靜態加密** | 建議 | **強制**（設定檔/金鑰庫加密）| SL3 + 硬體安全儲存 |
| **金鑰管理** | 基本儲存（非 hardcode）| 安全金鑰管理（輪替/撤銷）| **HSM 保護的金鑰 + 內部操作** |
| **資訊殘留** | — | 安全抹除 | 自動殘留清除 |

**常見不合格**：HTTP 明文、無 TLS、私鑰存一般 flash、無 TRNG（用 PRNG 生金鑰）

---

### FR5 (RDF) — 限制資料流

| 要求 | SL 2 | SL 3 | SL 4 |
|---|---|---|---|
| **預設策略** | **Default deny**（所有服務預設關閉）| 同左 | 同左 |
| **存取控制** | ACL (IP/port whitelist) | SL2 + 工業協定 **DPI** | SL3 + 實體隔離/Data diode |
| **最小暴露** | 服務不綁 0.0.0.0 | 同左 | 獨立 NIC 硬體隔離 |
| **文件** | 通訊矩陣（來源/目的/協定/port/方向）| 同左 | 同左 |

**常見不合格**：綁 0.0.0.0、pprof/debug port 全網開放、CORS `*`、缺通訊矩陣

---

### FR6 (TRE) — 事件回應

| 要求 | SL 2 | SL 3 | SL 4 |
|---|---|---|---|
| **事件記錄** | 安全相關事件記錄（登入/失敗/變更）| SL2 + **防篡改日誌 (HMAC)** | SL3 + forensic readiness |
| **匯出** | 可匯出 (syslog) | **即時匯出 (syslog over TLS)** | 同左 |
| **告警** | 本機告警 | 中央 SIEM 整合 | 自動化事件響應 (SOAR) |
| **時間同步** | 建議 | **強制 NTP** | 同左 |

**常見不合格**：安全事件沒記錄、無 NTP、無告警、日誌無防篡改

---

### FR7 (RA) — 資源可用性

| 要求 | SL 2 | SL 3 | SL 4 |
|---|---|---|---|
| **DoS 防護** | 抗 moderate DoS (rate limiting) | 抗 severe DoS | 抗 extreme DoS |
| **資源管理** | 基本資源監控 | 資源使用上限 + **安全功能不降級** | 自動資源調節 |
| **故障安全** | **Fail-safe** 設計（斷訊→安全狀態）| SL2 + 驗證過的 fail-safe | 自動安全復原 |
| **備援** | — | 可支援 standby | **Hot standby**（無感切換） |
| **斷電恢復** | 自動恢復到斷電前安全狀態 | SL2 + 完整性檢查 | 同左 |

**常見不合格**：無 rate limit → 遠端 DoS、安全功能 resource 不足時降級、watchdog 只監 CPU 不監應用

---

## 3. 組件類型 × SL 2 最低要求差異

| FR | Software App (SA) | Embedded Device (ED) | Host Device (HD) | Network Device (ND) |
|---|---|---|---|---|
| **FR1** | 使用者 auth + API token | 使用者 + 裝置憑證 | 使用者 + OS 帳號 + TPM | 管理介面 auth |
| **FR2** | RBAC (app level) | RBAC (firmware level) | OS DAC + app RBAC | 管理介面 RBAC |
| **FR3** | 輸入驗證 + 相依掃描 | **Secure Boot + FW 簽章** | UEFI Secure Boot + dm-verity | FW 簽章驗證 |
| **FR4** | TLS（倚賴 OS）| **TLS + SE 保護的私鑰** | TLS + 磁碟加密 | 管理介面 TLS |
| **FR5** | 最小 port + CORS 限制 | **Default deny + 綁定內部 IP** | OS firewall + VLAN | **核心功能：ACL/VLAN/DPI** |
| **FR6** | Structured logging | Syslog + NTP | Syslog + SIEM agent | Syslog + NetFlow |
| **FR7** | Rate limiting + graceful degradation | **Watchdog + fail-safe** | Kernel watchdog + cgroup | Control plane protection |

---

## 4. SL 2 達標的最低可操作清單

按優先序排列，做完這 10 項即達到 SL 2 骨架：

```
□ 1. FR1: 移除所有預設密碼，出廠強制首次修改
□ 2. FR1: API 加 token/JWT 認證（不裸奔）
□ 3. FR2: 建 RBAC（最少 operator/engineer/admin 三角色）
□ 4. FR2: 移除 DevMode 全繞過（compile out, 非 runtime flag）
□ 5. FR3: 韌體更新加 ED25519/RSA 簽章驗證
□ 6. FR3: 量產後鎖 JTAG/SWD（燒 fuse）
□ 7. FR4: 全鏈路上 TLS 1.2+
□ 8. FR5: 服務不綁 0.0.0.0，關閉 pprof/debug port
□ 9. FR6: 安全事件記錄 + NTP 同步
□ 10. FR7: API rate limiting + fail-safe（斷訊自動安全停機）
```

---

## 5. SL 2 → SL 3 的關鍵跳躍

三件事從 SL 2 升級到 SL 3 變化最大：

| 升級點 | SL 2 | SL 3 | 最難在哪 |
|---|---|---|---|
| **Secure Boot** | 建議 | **強制** | 需要硬體信任根（OTP/eFuse），不是純軟體能做到的 |
| **MFA + x509** | 密碼 + 簡單憑證 | **多因素 + mTLS** | 需要 PKI 基礎設施 + SE 儲存私鑰 |
| **金鑰管理** | 基本儲存 | **完整生命週期管理** | 金鑰輪替、撤銷機制、HSM 操作——組織流程的成熟度 |

> 如果你的產品**硬體設計還沒定案**：現在就預留 OTP/eFuse 給 public key hash、預留 I2C 介面給 SE、預留 external watchdog 腳位——這三件事在 PCB layout 之後就很難加了。

---

相關：[CONTEXT.md](../CONTEXT.md)、[FR 1-7 全景](01-foundations/04-foundational-requirements.md)
