# CVE-2026-31431 (Copy Fail) 弱點檢查 Playbook

## 概述

本 Ansible Playbook **僅進行檢查與報告**，不會主動修補或變更系統設定。

用於檢查 Ubuntu 22.04、Ubuntu 24.04 及 RHEL 8 系統是否受到 **CVE-2026-31431**（又稱 "Copy Fail"）弱點影響，並判斷 kmod / kernel 是否已更新至修補版本。

**CVE-2026-31431** 是 Linux kernel 中 `algif_aead` 模組（AF_ALG 加密介面）的本地權限提升弱點，CVSS 評分 7.8（HIGH）。未授權的本地使用者可透過此弱點在數秒內取得 root 權限。此弱點影響自 2017 年以來所有 Linux kernel 版本。

## 修補版本參考

### Ubuntu — kmod 套件修補版本

Ubuntu 的修補方式是透過更新 **kmod** 套件，在 modprobe 規則中封鎖 `algif_aead` 模組載入。

| 發行版 | 修補 kmod 版本 | 你的版本 | 狀態 |
|--------|---------------|----------|------|
| Ubuntu 22.04 (Jammy) | `29-1ubuntu1.1` | `29-1ubuntu1.1` | ✅ 已達修補版本 |
| Ubuntu 24.04 (Noble) | `31+20240202-2ubuntu7.2` | `31+20240202-2ubuntu7.1` | ❌ 未達修補版本 |

### Ubuntu — Kernel 版本

| 發行版 | 你的 Kernel 版本 | 說明 |
|--------|-----------------|------|
| Ubuntu 22.04 | `5.15.0-177-generic` | Kernel 本身仍標記為 Vulnerable，修補透過 kmod 實現 |
| Ubuntu 24.04 | `6.8.0-101-generic` | Kernel 本身仍標記為 Vulnerable，修補透過 kmod 實現 |

> **重要**：Ubuntu 的修補策略是透過 kmod 套件封鎖 `algif_aead` 模組，而非直接修補 kernel。只要 kmod 版本達到修補版本，即使 kernel 版本未變更，系統也已受到保護。

### RHEL 8 — Kernel 修補版本

| 發行版 | 修補 Kernel 版本 |
|--------|-----------------|
| RHEL 8 | `4.18.0-553.121.1.el8_10` |
| RHEL 9 | `5.14.0-611.49.2.el9_7` |

## 判斷結果摘要

| 系統 | 項目 | 結果 |
|------|------|------|
| Ubuntu 22.04 | kmod `29-1ubuntu1.1` | ✅ 已達修補版本 |
| Ubuntu 24.04 | kmod `31+20240202-2ubuntu7.1` | ❌ 未達修補版本（需 `31+20240202-2ubuntu7.2`）|
| Ubuntu 22.04 | Kernel `5.15.0-177-generic` | ⚠️ Kernel 仍為 Vulnerable（但 kmod 已修補即安全）|
| Ubuntu 24.04 | Kernel `6.8.0-101-generic` | ⚠️ Kernel 仍為 Vulnerable（kmod 未達修補版本）|

## 使用方式

```bash
ansible-playbook -i inventory main.yml
```

> Playbook 僅檢查並輸出報告，不會對系統做任何變更。

## Playbook 檢查項目

1. **kmod 版本檢查** — 比對已安裝的 kmod 版本是否達到修補版本
2. **Kernel 版本檢查** — 比對 RHEL 系統的 kernel 是否達到修補版本
3. **algif_aead 模組狀態** — 檢查模組是否已載入
4. **Kernel config** — 檢查 `CONFIG_CRYPTO_USER_API_AEAD` 設定
5. **Boot cmdline 緩解** — 檢查是否已有 blacklist 參數
6. **modprobe.d 緩解** — 檢查是否已有 modprobe 封鎖規則
7. **AF_ALG socket 測試** — 實際測試是否能建立 AF_ALG socket

## 風險等級分類

| 等級 | 說明 |
|------|------|
| `not_affected` | `CONFIG_CRYPTO_USER_API_AEAD` 未啟用，不受影響 |
| `mitigated` | 已透過 boot 參數或 modprobe 規則封鎖 |
| `high` | algif_aead 模組已載入或為 built-in，高風險 |
| `medium` | 模組存在且可載入，中風險 |
| `low` | AF_ALG socket 無法建立，低風險 |

## 參考資料

- [Ubuntu CVE-2026-31431 公告](https://ubuntu.com/security/CVE-2026-31431)
- [Red Hat RHSB-2026-02](https://access.redhat.com/security/vulnerabilities/RHSB-2026-02)
- [NVD CVE-2026-31431](https://nvd.nist.gov/vuln/detail/CVE-2026-31431)
- [Ubuntu Discourse — 修補公告](https://discourse.ubuntu.com/t/fixes-available-for-cve-2026-31431-copy-fail-linux-kernel-local-privilege-escalation-vulnerability/81498)

---

*最後更新：2026-05-06*
