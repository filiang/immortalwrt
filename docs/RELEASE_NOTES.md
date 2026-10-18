# R9000 (ImmortalWrt / alpine) — 2026-10-18 快照

## 概要

- 内核 **6.18.52**，ImmortalWrt SNAPSHOT（alpine/generic → netgear_r9000）
- 无线驱动切换为**上游 ath10k + 上游固件**（非 ath10k-ct）
- **5G 默认关闭**（实测不可用，详见下文）；2.4G 默认开启
- 60GHz 默认关闭，SSID 固定为 `R9000-60G`
- 默认 LAN 地址 **10.0.0.1/24**（避开光猫常见 192.168.1.1 网段）
- 固件：`factory-20261018-1955-chainmask-bp.img`（61,866,113 B）
- sha256：`74f01683725b87c6a42bac65661c12bc6afb88fec3f0ebb07cc9b5376d4a5782`

## 5G 结论（默认关闭）

实测数据：

| 项目 | 结果 |
|---|---|
| 链0 / 链1 RSSI | **-75 / -53 dBm**（相差约 22 dB） |
| 下行协商 | **MCS1 / NSS1**（65 Mbps PHY） |
| 上行协商 | MCS4 / NSS2（390 Mbps PHY） |
| 下行重传率 | **约 158%** |
| 实测吞吐 | 下行约 **36 Mbps**，上行约 124 Mbps |
| 稳定性 | 偶发断线 |

- 原厂 QCA NSS 专有驱动可跑 **1733 Mbps（4×4）**，说明射频硬件正常，属 ath10k 驱动/校准层差异
- 已在 `99-wifi-preset` 中默认关闭 5G 射频与接口，保留 `channel=149` / `VHT80` 参数
- 手动启用：

```sh
uci set wireless.radio1.disabled='0'
uci set wireless.default_radio1.disabled='0'
uci commit wireless
wifi reload
```

## 其他改动

- **ath10k chainmask 补丁**：`package/kernel/mac80211/patches/ath10k/999-ath10k-limit-chainmask.patch`（backports，链掩码限制为 0x3）
- **board-2.bin**：用原厂 `boardData_QCA9984_CUS239_5G_v1_001.bin` / `CUS260_2G_v1_002.bin` 替换对应条目（各 12,064 B）
- **DTS 预校准回滚**：5G → `precal_1000`、2.4G → `precal_5000`
- **2.4G**：HT40 预设（实际可能被 20/40 共存回退为 20 MHz）
- **60G**：默认关闭，接口 `60GHz-radio0`，SSID `R9000-60G`；**尚未实测**，稳定性未知
- **默认网络**：LAN 10.0.0.1/24，Web `http://10.0.0.1`，默认 root 无密码（请首次登录后立即设置）
- **实测日志**：脱敏后收录于 `docs/logs/`

## 已知问题

- 5G 链0 偏弱，下行只能 2×2→NSS1 回退，默认关闭
- 2.4G 可能停留在 20 MHz（20/40 共存）
- 10G SFP+ 默认不启用

## 升级提示

- 从旧版本升级会重置 `/etc/openclash/config/`，需重新导入订阅
- 5G 若需临时启用，按上文命令操作；不建议长期使用
