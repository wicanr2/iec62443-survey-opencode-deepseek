# Docker 部署安全：如何達到 IEC 62443-4-2 SL2

Docker 本身不提供安全——它提供封裝。要讓容器化部署滿足 IEC 62443-4-2 SL2 的 FR5（限制資料流），必須刻意設定網路隔離、存取控制和最小暴露。這篇把 Docker 的安全基元對應到 Zone & Conduit 模型，給出可操作的配置。

這篇不是自說自話。每條建議後面都標了外部參考來源。

## Docker 網路模型 vs Zone & Conduit

IEC 62443 的安全模型是空間的：把不同信任等級的資產放進不同 Zone，Zone 之間只透過受控的 Conduit 通訊。Docker 的網路模型跟這個結構驚人地對齊——只是多數人沒刻意用它。

| Docker 概念 | Zone & Conduit 對應 | 預設行為 | SL2 要求 |
|---|---|---|---|
| Bridge network (default) | 一個 Zone | 所有容器互聯（`--icc=true`），無隔離 | **必須關閉 ICC 或用 custom network** |
| Custom bridge network | 一個 Zone | 僅同 network 的容器互通 | 精確配置，只讓需要的容器在同一個 network |
| `ports:` (host binding) | Conduit | `0.0.0.0:PORT` → 全介面開放 | **只讓 reverse proxy 有 ports:，且應綁定特定 IP** |
| `expose:` | Zone 內通訊 | 僅 Docker 內網可見 | 內部服務預設使用 |
| Docker socket | Conduit 端點 | 無保護的 root 等價存取 | **絕不掛進容器；若需遠端，強制 TLS** |

參考來源：[OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html) Rule #1, #5, #8；[Docker 官方網路文件](https://docs.docker.com/network/)；[CIS Docker Benchmark v1.8.0](https://www.cisecurity.org/benchmark/docker) §3

## SL2 的最低要求

IEC 62443-4-2 的 FR5 在 SL2 有四條核心要求。下面逐條給出 Docker 的對應配置。

### FR5-1：Default Deny（預設拒絕）

標準說：所有服務/port 出廠預設關閉，只開放被明示允許的。

Docker 配置：

```yaml
# /etc/docker/daemon.json
{
  "icc": false,              # 關閉容器間通訊（預設是 true）
  "userland-proxy": false,   # 減少攻擊面
  "iptables": true           # 保留 iptables 規則以配合 host firewall
}
```

`icc: false` 把 Docker 的預設從「容器全部互聯」變成「預設全部隔離」。之後每個需要互通的容器，必須放在同一個 custom network 上才能通——這就是 default deny。

參考來源：[Docker daemon configuration](https://docs.docker.com/engine/reference/commandline/dockerd/)；[CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker) §3.1

### FR5-2：ACL 與最小暴露

標準說：IP/port whitelist，服務不綁 0.0.0.0。

Docker 配置——拆多個 network：

```yaml
# docker-compose.yml
networks:
  frontend:        # nginx ↔ backend
    driver: bridge
    internal: false
  backend:         # backend ↔ mongo/redis
    driver: bridge
    internal: true    # 不允許對外，純內網
  isolated:        # 完全隔離
    driver: bridge
    internal: true

services:
  nginx:
    networks: [frontend]
    ports:
      - "10.0.1.5:443:443"    # 只綁特定 host IP，不是 0.0.0.0

  backend:
    networks: [frontend, backend]  # 可以跟 nginx 和 DB 通
    expose: ["8081"]               # 不映射到 host
    # 應用層也限制
    environment:
      - LISTEN_ADDR=0.0.0.0:8081  # 容器內 0.0.0.0 可接受（因為 network 已隔離）

  mongo:
    networks: [backend]            # 只在 backend network 上
    expose: ["27017"]
    # mongo 只聽 backend network 上的介面
    command: ["mongod", "--bind_ip", "mongo"]  # 只綁 Docker DNS hostname

  redis:
    networks: [backend]
    expose: ["6379"]
```

mongo 和 redis 只存在於 `backend` network 上。如果攻擊者拿到 nginx 容器的 shell，它看不到 mongo——因為 nginx 不在 `backend` network 上。這就是 Zone 隔離。

`internal: true` 確保這個 network 上的容器**無法對外連線**，只能跟同 network 的容器通。適合 backend network（DB 不需要連 Internet）。

參考來源：[Docker Container Networking](https://docs.docker.com/network/)；[Docker Compose network configuration](https://docs.docker.com/compose/compose-file/compose-file-v3/#networks)；[OWASP Rule #5: Inter-Container Connectivity](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html#rule-5-enable-docker-content-trust)

### FR5-3：通訊埠與服務最小化

標準說：只啟用必要的服務，不必要的全部關閉。

Docker 容器層的資源限制與 capability 管控：

```yaml
services:
  backend:
    security_opt:
      - no-new-privileges:true      # 禁止提權
    cap_drop:
      - ALL                          # 移除所有 capabilities
    cap_add:
      - NET_BIND_SERVICE             # 只加回綁定 port 需要的
    read_only: true                  # 根檔案系統唯讀
    tmpfs:
      - /tmp                         # 只開放需要寫入的目錄
    deploy:
      resources:
        limits:
          cpus: "2"
          memory: 512M               # 防止單一容器耗盡 host 資源 (FR7)
```

參考來源：[Docker Engine Security](https://docs.docker.com/engine/security/)；[NIST SP 800-190 Application Container Security Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf) §4.3

### FR5-4：通訊矩陣文件化

IEC 62443-4-2 P8 (SG) 要求 hardening guide 載明通訊矩陣。Docker 部署的通訊矩陣長這樣：

| 來源容器 | 目的容器 | 協定/Port | 方向 | Network | 用途 |
|---|---|---|---|---|---|
| nginx | backend | HTTP/8081 | → | frontend | API 代理 |
| backend | mongo | TCP/27017 | ↔ | backend | 資料讀寫 |
| backend | redis | TCP/6379 | ↔ | backend | 快取 |
| host | nginx | HTTPS/443 | → | host | 外部入口 |
| (any) | docker.sock | — | 禁止 | — | 不允許 |

這張表是 Conduit 設計的基礎——整合商或資安稽核員需要它來判斷「這些容器之間的通道是否都受控」。

參考來源：[IEC 62443-4-2 §P8 Security Guidelines](https://webstore.iec.ch/en/publication/34421)；[Cisco ISA/IEC-62443-3-3 Whitepaper](https://www.cisco.com/c/en/us/products/collateral/security/isaiec-62443-3-3-wp.html)

## 容器 vs 實體 Zone：Docker 能到 SL 幾？

一個常見問題：純 Docker 的網路隔離能滿足哪個 SL？

| SL 等級 | Docker 能做到嗎？ | 說明 |
|---|---|---|
| SL 1 | ✅ 是 | 基本 port 設定、服務最小化 |
| SL 2 | ✅ 是（需要刻意配置） | Default deny + custom bridge network + `icc: false` + 通訊矩陣 |
| SL 3 | △ 部分（需要 host 層輔助） | 工業協定 DPI 需要 host 層的 iptables/nftables 或外部 firewall。Docker 本身不做 L7 深度檢測 |
| SL 4 | ❌ 不行 | 需要實體隔離、Data diode。Docker bridge network 是 Linux kernel 的軟體隔離，做不到物理保證 |

> Docker 的網路隔離本質是 Linux kernel namespace + iptables。對 SL2 的威脅模型（一般駭客，簡單手段）是夠的。但對 SL3（針對性攻擊，OT 專精），攻擊者可能利用 kernel vulnerability 跨越 namespace——此時需要 host 層或實體隔離來補償（CCSC 2）。

參考來源：[IEC 62443 Zone & Conduit 實務指南](https://www.cagripolat.com/iec62443/en/iec-62443-zone-conduit-design-ot-practical-guide)；[MDPI: Security Aspects of Zones and Conduits](https://www.mdpi.com/2624-800X/6/2/52)

## 半自動化合規檢查

SL2 的要求可以部分自動化驗證，不必全手動稽核。

| 工具 | 檢查什麼 | 對應 FR5 |
|---|---|---|
| [docker-bench-security](https://github.com/docker/docker-bench-security) | CIS Docker Benchmark 自動掃描（含 ICC、socket 暴露、privileged container） | FR5-1, FR5-2 |
| [dockerd 直接檢查](https://docs.docker.com/engine/security/) | `docker info | grep -i icc` 確認 ICC 關閉 | FR5-1 |
| [OPA/Conftest](https://www.conftest.dev) | Rego policy 檢查 docker-compose.yml（不該有 `ports:` 的服務、不該有 `privileged: true`） | FR5-1, FR5-2, FR5-3 |
| [Trivy](https://github.com/aquasecurity/trivy) | 容器 image CVE 掃描 | FR3 (SI) → FR5 相依 |

最基本的 CI 檢查（放進 `.github/workflows/` 或 Makefile）：

```bash
# 檢查 docker-compose 中不該對外暴露的服務
grep -B1 'ports:' docker-compose.yml | grep -v nginx && echo "FAIL: non-nginx service has ports:"

# 確認 daemon ICC 關閉
docker info | grep -q "ICC: false" || echo "FAIL: ICC not disabled"

# 掃描 image 弱點
trivy image --severity HIGH,CRITICAL myapp:latest
```

參考來源：[Docker Bench for Security](https://github.com/docker/docker-bench-security)；[Conftest](https://www.conftest.dev)；[OPA Docker Authorization](https://www.openpolicyagent.org/docs/latest/docker-authorization/)

## 總結：SL2 Docker 部署檢查表

```
□ 1. daemon.json: "icc": false（關閉容器間通訊）
□ 2. 只有 nginx/traefik 有 ports:，其餘服務用 expose:
□ 3. ports: 綁定具體 host IP（非 0.0.0.0）
□ 4. 拆多個 custom bridge network，DB 放 internal network
□ 5. /var/run/docker.sock 不掛進任何容器
□ 6. 所有容器：cap_drop: ALL + no-new-privileges
□ 7. 根檔案系統 read_only（需要寫入的目錄用 tmpfs）
□ 8. 產出通訊矩陣文件
□ 9. CI 中跑 docker-bench-security + Trivy
```

做完這 9 項，Docker 部署層的 FR5 就達到 SL2。

---

## 本文使用縮寫對照

| 縮寫 | 全稱 | 說明 |
|---|---|---|
| ACL | Access Control List | 存取控制清單 |
| CIS | Center for Internet Security | 網路安全中心，發布安全基準 |
| FR5 | Foundational Requirement 5 | 限制資料流 (RDF) |
| ICC | Inter-Container Communication | Docker 容器間通訊 |
| RDF | Restricted Data Flow | 限制資料流 |
| SL2 | Security Level 2 | 防一般駭客的蓄意攻擊 |
| SR | System Requirement | 系統安全需求 (IEC 62443-3-3) |

## 外部參考文獻

1. [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html) — 14 條 Docker 安全鐵則
2. [CIS Docker Benchmark v1.8.0](https://www.cisecurity.org/benchmark/docker) — Docker 安全配置基準
3. [Docker Bench for Security](https://github.com/docker/docker-bench-security) — CIS Benchmark 自動檢查腳本
4. [NIST SP 800-190: Application Container Security Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf) — NIST 容器安全指南
5. [Docker Container Networking (官方)](https://docs.docker.com/network/) — 網路驅動與容器間通訊
6. [Docker Engine Security (官方)](https://docs.docker.com/engine/security/) — Daemon 安全架構
7. [IEC 62443 Zones and Conduits Explained](https://iotworlds.com/iec-62443-zones-and-conduits-explained-with-a-practical-plant-example/) — 工廠實例
8. [Cisco ISA/IEC-62443-3-3 Compliance Whitepaper](https://www.cisco.com/c/en/us/products/collateral/security/isaiec-62443-3-3-wp.html) — Zone/Conduit 技術對照
9. [MDPI: Security Aspects of Zones and Conduits in IEC 62443](https://www.mdpi.com/2624-800X/6/2/52) — 學術論文
10. [OPA / Conftest Docker Policy](https://www.conftest.dev) — Policy-as-code 檢查 docker-compose
11. [IEC 62443 Zone Conduit Design in OT](https://www.cagripolat.com/iec62443/en/iec-62443-zone-conduit-design-ot-practical-guide) — OT 實務指南

> [完整術語表](../CONTEXT.md)
