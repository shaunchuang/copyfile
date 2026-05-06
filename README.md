# CVE-2026-31431 (Copy Fail) 弱點檢查 Playbook

## 概述

本 Ansible Playbook **僅進行檢查與報告**，不會主動修補或變更系統設定。

執行後產生 `cve_2026_31431_report.csv`，可直接用 Excel 開啟，一目了然。

## 使用方式

```bash
ansible-playbook -i inventory main.yml
```

## CSV 輸出範例

| 主機名稱 | IP | 作業系統 | 目前 Kernel | 目前 kmod | 修補方式 | 需要版本 | 是否已修補 | algif_aead 狀態 | 緩解措施 | 風險等級 |
|----------|-----|---------|-------------|-----------|---------|---------|-----------|----------------|---------|---------|
| web01 | 10.0.1.10 | Ubuntu 22.04 | 5.15.0-177-generic | 29-1ubuntu1.1 | kmod 套件更新 | kmod >= 29-1ubuntu1.1 | 是 | module 未載入 | modprobe 封鎖 | patched |
| web02 | 10.0.1.11 | Ubuntu 24.04 | 6.8.0-101-generic | 31+20240202-2ubuntu7.1 | kmod 套件更新 | kmod >= 31+20240202-2ubuntu7.2 | 否 | module 未載入 | 無 | medium |
| db01 | 10.0.2.20 | RedHat 8.10 | 4.18.0-553.109.1 | 28-7.el8_10 | kernel 更新 | kernel >= 4.18.0-553.121.1.el8_10 | 否 | built-in (已編入核心) | 無 | high |

## 風險等級定義

| 等級 | 意義 | 說明 |
|------|------|------|
| `patched` | ✅ 已修補 | 已命中已知 kernel / kmod 修補版本 |
| `mitigated` | 🛡️ 已緩解 | 已有 kmod / boot / modprobe mitigation，且 algif_aead 未載入 |
| `not_affected` | ✅ 不受影響 | CONFIG_CRYPTO_USER_API_AEAD is not set |
| `high` | 🔴 高風險 | algif_aead 已載入，或 CONFIG=y 且未確認修補 |
| `medium` | 🟡 中風險 | module 存在 (config=m)，可被載入利用 |
| `low` | 🟢 低風險 | AF_ALG socket 被阻擋 |
| `unknown` | ⚠️ 未知 | 資訊不足，需人工確認 vendor patch 狀態 |

## 各發行版修補方式差異

| 發行版 | 修補方式 | 說明 |
|--------|---------|------|
| **Ubuntu 22.04 / 24.04** | 更新 **kmod** 套件 | kmod 更新後會在 modprobe.d 加入 `install algif_aead /bin/false`，封鎖模組載入。Kernel 本身不需更新。 |
| **RHEL 8 / 9** | 更新 **kernel** | RHEL 的 algif_aead 是 built-in (`=y`)，modprobe 封鎖無效。必須更新 kernel 或加 boot 參數 `initcall_blacklist=algif_aead_init`。 |

## 已確認的修補版本

### Ubuntu (kmod)

| 發行版 | 修補 kmod 版本 | 來源 |
|--------|---------------|------|
| Ubuntu 20.04 (Focal) | `27-1ubuntu2.1+esm1` | Ubuntu Discourse 公告 |
| Ubuntu 22.04 (Jammy) | `29-1ubuntu1.1` | Ubuntu Discourse 公告 |
| Ubuntu 24.04 (Noble) | `31+20240202-2ubuntu7.2` | Ubuntu Discourse 公告 |

### RHEL (kernel)

| 發行版 | 修補 Kernel 版本 | 來源 |
|--------|-----------------|------|
| RHEL 8 | `4.18.0-553.121.1.el8_10` | RHSB-2026-02 / CIQ KB |
| RHEL 9 | `5.14.0-611.49.2.el9_7` | RHSB-2026-02 |

## 參考資料

- [Ubuntu Discourse — 修補公告](https://discourse.ubuntu.com/t/fixes-available-for-cve-2026-31431-copy-fail-linux-kernel-local-privilege-escalation-vulnerability/81498)
- [Red Hat RHSB-2026-02](https://access.redhat.com/security/vulnerabilities/RHSB-2026-02)
- [CIQ KB — Rocky Linux 8/9 Mitigation](https://kb.ciq.com/article/rocky-linux/rl-cve-2026-31431-mitigation)
- [NVD CVE-2026-31431](https://nvd.nist.gov/vuln/detail/CVE-2026-31431)

---

*最後更新：2026-05-06*
