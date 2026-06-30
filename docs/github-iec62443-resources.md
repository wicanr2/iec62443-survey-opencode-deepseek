# IEC 62443 GitHub 生態資源彙整

> 2026-06-30 以 WebSearch 從 GitHub 搜尋彙整。涵蓋合規工具、審計框架、參考架構、教育資源、測試工具、Secure Boot 實作等 60+ repos。

---

## 一、合規檢查表 / 審計工具

| Repo | ★ | 說明 | 技術 |
|---|---|---|---|
| [isecwire/iec62443-audit](https://github.com/isecwire/iec62443-audit) | 0 | IEC 62443-3-3 + 4-2 互動式評估，含 gap analysis、TUI、HTML/JSON/CSV 匯出 | Python |
| [icscheck-tool/icscheck](https://github.com/icscheck-tool/icscheck) | 4 | ICS/SCADA 單機審計：36+ security controls 對照 FR1-7 & NIS2，WinCC 整合，HTML 報告 | PowerShell |
| [leaberg69/aem-iec62443-checklist](https://github.com/leaberg69/aem-iec62443-checklist) | 0 | 4-2 SL2 的 47 項 yes/no 驗收清單 + Python validator | Python |
| [frangelbarrera/ICS-Cybersecurity-Audit](https://github.com/frangelbarrera/ICS-Cybersecurity-Audit) | 15 | 5-phase ICS/OT 審計框架，對照 62443+NIST 800-82+MITRE ATT&CK | Python |
| [ynotbhatc/rego_policy_libraries](https://github.com/ynotbhatc/rego_policy_libraries) | 10 | 464 條 OPA Rego 政策，涵蓋全 51 SRs (FR1-7, SL1-4)，可入 CI/CD | OPA Rego |
| [ninjat6/IEC-62443-conformity-analysis](https://github.com/ninjat6/IEC-62443-conformity-analysis) | 1 | 62443 合規性分析工具 | Python |

---

## 二、GAP 分析 / 風險評估

| Repo | ★ | 說明 | 技術 |
|---|---|---|---|
| [MCYP-UniversidadReyJuanCarlos/19-20_frripe](https://github.com/MCYP-UniversidadReyJuanCarlos/19-20_frripe) | 2 | IEC 62443 方法論量化風險分析，計算 Target SL | VBA |
| [grandMa5ter/IEC62443_Dashboard](https://github.com/grandMa5ter/IEC62443_Dashboard) | 1 | Web dashboard 計算 Zone SL-T vs SL-C 對比 | HTML |
| [cyberriskcanvas/cyberriskcanvas-app](https://github.com/cyberriskcanvas/cyberriskcanvas-app) | 1 | 自部署威脅建模+TARA (62443 / EU CRA / NIS-2)，STRIDE/SBOM/CVE triage | TypeScript |
| [arnavparekar/ics-zoneaudit](https://github.com/arnavparekar/ics-zoneaudit) | 0 | Nmap→Purdue Model zone 自動分類 + 15 項 hardening checklist + conduit violation 偵測 | Python |
| [dakhasuresh/ot-asset-classifier](https://github.com/dakhasuresh/ot-asset-classifier) | 0 | 62443 對齊 OT 資產分類器 + T×V×I 風險評分 | Python |

---

## 三、參考架構 (Reference Architecture)

| Repo | ★ | 說明 | 技術 |
|---|---|---|---|
| [rick-rami94/manufacturing-security-reference-architecture](https://github.com/rick-rami94/manufacturing-security-reference-architecture) | 0 | 製造業 OT/ICS 安全參考架構，對照 62443 + NIST 800-82r3 + CIS v8.1 | Mermaid |
| [rick-rami94/logistics-security-reference-architecture](https://github.com/rick-rami94/logistics-security-reference-architecture) | 0 | 24/7 物流配銷中心 IT/OT 分割參考架構 | Mermaid |
| [elvis12234/Industrial-Network-Architecture-](https://github.com/elvis12234/Industrial-Network-Architecture-) | 0 | ISA/IEC 62443 合規 Purdue Model 工控網路架構 | 文件 |

---

## 四、教育 / 訓練資源

| Repo | ★ | 說明 | 技術 |
|---|---|---|---|
| [SecurityWithAdarsh/OT-ICS-Security-Notes](https://github.com/SecurityWithAdarsh/OT-ICS-Security-Notes) | 7 | 完整 OT/ICS 安全筆記：SCADA/PLC/DCS、IEC 62443、NIST CSF、攻擊技術 | Markdown |
| [JHaimaka/CRA-IEC62443-SBOM](https://github.com/JHaimaka/CRA-IEC62443-SBOM) | 0 | 芬蘭開放訓練教材：EU CRA + IEC 62443 + SBOM (Syft/Grype) | 文件 |
| [jeff-witt/manufacturing-ot-resources](https://github.com/jeff-witt/manufacturing-ot-resources) | 0 | 精選製造業 IT/OT 工具/標準/框架資源清單 | Markdown |
| [leaberg69/awesome-industrial-modbus](https://github.com/leaberg69/awesome-industrial-modbus) | 1 | Awesome list: Modbus 資源+工具+裝置，含 62443 安全章節 | Markdown |
| [dawuds/OT-Security](https://github.com/dawuds/OT-Security) | 5 | OT 合規瀏覽器，對照 62443 / NIST 800-82 / MITRE ATT&CK | CSS |

---

## 五、Lab / 測試平台

| Repo | ★ | 說明 | 技術 |
|---|---|---|---|
| [michailidimaria/Virtual-Lab-ICS-Segmentation](https://github.com/michailidimaria/Virtual-Lab-ICS-Segmentation) | 0 | VirtualBox + OPNsense 多區隔工業環境 lab (Purdue Model) | VirtualBox |
| [gammahazard/harvester-os-portfolio](https://github.com/gammahazard/harvester-os-portfolio) | 1 | 實體 OT/ICS 測試平台：Siemens S7-1200 + RevPi + Raspberry Pi | Python |
| [SivaramRajaRajasekaran/OT-Security-Portfolio](https://github.com/SivaramRajaRajasekaran/OT-Security-Portfolio) | 0 | Docker 化 IIoT 資安 lab：MQTT 攻擊模擬 + 62443 協定分析 | Python |
| [whosdaniel/iec62443-multitenant-testbed](https://github.com/whosdaniel/iec62443-multitenant-testbed) | 0 | Docker testbed 重現 62443 多租戶監控盲點 | Python |

---

## 六、安全工具 / 平台

| Repo | ★ | 說明 | 技術 |
|---|---|---|---|
| [SiteQ8/OpenICS-Atlas](https://github.com/SiteQ8/OpenICS-Atlas) | 2 | ICS/OT 曝險情報平台：8 協定、Purdue 模型、72 項控制、Shodan 整合 | TypeScript |
| [SiteQ8/ConduitShield](https://github.com/SiteQ8/ConduitShield) | 0 | OT/IIoT 工具：資產發現、SBOM、zone&conduit 政策、62443 合規 | — |
| [SiteQ8/ics-iot-ot-hardening](https://github.com/SiteQ8/ics-iot-ot-hardening) | 12 | OT 硬體化平台：資產掃描、弱點掃描、62443 / NIST 800-82 對齊 | Python |

---

## 七、ICS 安全測試框架 (間接相關，但工控安全必知)

| Repo | ★ | 說明 |
|---|---|---|
| [dark-lbp/isf](https://github.com/dark-lbp/isf) | 1,105 | ICS Exploitation Framework，工業控制系統滲透測試 |
| [nsacyber/GRASSMARLIN](https://github.com/nsacyber/GRASSMARLIN) | 1,059 | NSA 開發的 ICS/SCADA 被動網路拓撲發現工具 |
| [cisagov/Malcolm](https://github.com/cisagov/Malcolm) | 2,446 | CISA 開發的 ICS 網路流量分析套件 (Zeek/Suricata/Arkime) |
| [mushorg/conpot](https://github.com/mushorg/conpot) | 1,493 | ICS/SCADA 低互動 Honeypot (Modbus/S7/BACnet 等) |
| [digitalbond/Quickdraw-Suricata](https://github.com/digitalbond/Quickdraw-Suricata) | 52 | ICS IDS 規則集 (Modbus/DNP3/EtherNetIP) |
| [momalab/ICSREF](https://github.com/momalab/ICSREF) | 184 | ICS 韌體逆向工程工具 |

---

## 八、協定安全工具 (Modbus / DNP3 / S7)

| Repo | ★ | 說明 |
|---|---|---|
| [pymodbus-dev/pymodbus](https://github.com/pymodbus-dev/pymodbus) | 2,717 | Python Modbus 協定實作 (RTU/TCP) |
| [dnp3/opendnp3](https://github.com/dnp3/opendnp3) | 332 | DNP3 協定棧 (C++/Java/.NET) |
| [gijzelaerr/python-snap7](https://github.com/gijzelaerr/python-snap7) | 812 | Python S7 通訊 (Siemens PLC) |
| [AlixAbbasi/Modbus-Fuzzer](https://github.com/AlixAbbasi/Modbus-Fuzzer) | 50 | Modbus 協定 Fuzzer |
| [ValtteriL/OpalOPC](https://github.com/ValtteriL/OpalOPC) | 9 | OPC UA 安全掃描器 |

---

## 九、嵌入式 Secure Boot 實作

| Repo | ★ | 說明 |
|---|---|---|
| [mcu-tools/mcuboot](https://github.com/mcu-tools/mcuboot) | 1,943 | 32-bit MCU Secure Bootloader (Zephyr/Mynewt) |
| [wolfSSL/wolfBoot](https://github.com/wolfSSL/wolfBoot) | 503 | 可攜式 OS 無關 MCU Secure Bootloader + OTA |
| [Foxboron/sbctl](https://github.com/Foxboron/sbctl) | 2,198 | UEFI Secure Boot 金鑰管理器 (Linux) |

---

## 十、IEC 62443-4-2 直接實作

| Repo | ★ | 說明 |
|---|---|---|
| [neutrinoguy/VoidProbe](https://github.com/neutrinoguy/VoidProbe) | 0 | Linux 嵌入式設備 4-2 DRRM (DoS/資源管理) 即時監控 | Go |
| [Harsh-467/Secure-iiot](https://github.com/Harsh-467/Secure-iiot) | 0 | 基於 4-2 的 IIoT 安全控制實作 | Python |
| [ValtteriL/iec62443-svv-demo](https://github.com/ValtteriL/iec62443-svv-demo) | 3 | 4-1 安全驗證與確認自動化 | PHP |

---

## 十一、形式化驗證

| Repo | ★ | 說明 |
|---|---|---|
| [kuliktomas/iec62443verification](https://github.com/kuliktomas/iec62443verification) | 0 | TLA+ 形式化驗證 ICS 是否符合 4-3-3 | TLA+ |

---

## 生態觀察

- IEC 62443 在 GitHub 上仍屬**小眾領域**：最高 star 僅 15（合規工具）、大部分 repo 為 2025-2026 新建
- **合規檢查表/審計腳本**佔最多數，但**開源安全控制實作**極少（多為文件/框架）
- ICS 安全測試生態非常成熟：ISF、GRASSMARLIN、Malcolm、conpot 為四大基石
- Secure Boot 首推 mcuboot（MCU）和 wolfBoot（跨平台）
- 政策即程式碼 (OPA Rego) 是新趨勢：`rego_policy_libraries` 將 51 條 SR 轉為可自動執行的政策

---

> 挑選標準：排除空 repo / 僅 README stub / 純 fork 無修改。優先收錄有程式碼或有實質文件者。
