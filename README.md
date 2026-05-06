# CVE-2026-31431 (Copy Fail) 弱點檢查 Playbook

## 概述

本 Ansible Playbook **僅進行檢查與報告**，不會主動修補或變更系統設定。

用於檢查 Ubuntu 22.04、Ubuntu 24.04 及 RHEL 8 系統是否受到 **CVE-2026-31431**（又稱 "Copy Fail"）弱點影響，並將結果輸出為 CSV 檔案。

**CVE-2026-31431** 是 Linux kernel 中 `algif_aead` 模組（AF_ALG 加密介面）的本地權限提升弱點，CVSS 評分 7.8（HIGH）。未授權的本地使用者可透過此弱點在數秒內取得 root 權限。此弱點影響自 2017 年以來所有 Linux kernel 版本。

## 使用方式

```bash
ansible-playbook -i inventory main.yml
```

執行完成後會在當前目錄產生 `cve_2026_31431_report.csv`，可直接用 Excel 開啟。

## 輸出 CSV 欄位說明

| 欄位 | 說明 |
|------|------|
| Host | 主機名稱 |
| Distribution | 發行版及版本 |
| Kernel | 執行中的 kernel 版本 |
| kmod_version | 已安裝的 kmod 套件版本 |
| kmod_patched | kmod 是否已達修補版本（True/False/N/A）|
| kernel_patched | kernel 是否已達修補版本（True/False/N/A）|
| algif_aead_loaded | algif_aead 模組是否已載入 |
| af_alg_loaded | af_alg 模組是否已載入 |
| module_file_exists | algif_aead .ko 檔案是否存在 |
| config_aead | kernel config 中 CRYPTO_USER_API_AEAD 設定 |
| boot_mitigated | boot 參數是否已有 blacklist |
| modprobe_mitigated | modprobe.d 是否已有封鎖規則 |
| af_alg_socket | AF_ALG socket 是否可建立 |
| risk_level | 風險等級判定 |

## 風險等級定義

| 等級 | 說明 |
|------|------|
| `patched` | 已命中已知 kernel / kmod 修補版本條件 |
| `mitigated` | 已有 kmod / boot / modprobe mitigation，且 algif_aead 未載入 |
| `not_affected` | `CONFIG_CRYPTO_USER_API_AEAD is not set`，不受影響 |
| `high` | algif_aead 已載入，或 `CONFIG_CRYPTO_USER_API_AEAD=y` 且未確認修補 |
| `medium` | module 存在、config=m、AF_ALG socket 可建立 |
| `low` | AF_ALG socket 被阻擋 |
| `unknown` | 資訊不足，需人工確認 vendor patch 狀態 |

## 修補版本參考

### Ubuntu — kmod 套件修補版本

| 發行版 | 修補 kmod 版本 | 你的版本 | 狀態 |
|--------|---------------|----------|------|
| Ubuntu 22.04 (Jammy) | `29-1ubuntu1.1` | `29-1ubuntu1.1` | ✅ 已達修補版本 |
| Ubuntu 24.04 (Noble) | `31+20240202-2ubuntu7.2` | `31+20240202-2ubuntu7.1` | ❌ 未達修補版本 |

### Ubuntu — Kernel 版本

| 發行版 | 你的 Kernel 版本 | 說明 |
|--------|-----------------|------|
| Ubuntu 22.04 | `5.15.0-177-generic` | 修補透過 kmod 實現，非 kernel 更新 |
| Ubuntu 24.04 | `6.8.0-101-generic` | 修補透過 kmod 實現，非 kernel 更新 |

### RHEL — Kernel 修補版本

| 發行版 | 修補 Kernel 版本 |
|--------|-----------------|
| RHEL 8 | `4.18.0-553.121.1.el8_10` |
| RHEL 9 | `5.14.0-611.49.2.el9_7` |

## 參考資料

- [Ubuntu CVE-2026-31431 公告](https://ubuntu.com/security/CVE-2026-31431)
- [Red Hat RHSB-2026-02](https://access.redhat.com/security/vulnerabilities/RHSB-2026-02)
- [NVD CVE-2026-31431](https://nvd.nist.gov/vuln/detail/CVE-2026-31431)
- [Ubuntu Discourse — 修補公告](https://discourse.ubuntu.com/t/fixes-available-for-cve-2026-31431-copy-fail-linux-kernel-local-privilege-escalation-vulnerability/81498)

---

*最後更新：2026-05-06*
