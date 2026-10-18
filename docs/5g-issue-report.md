# R9000 5G 无线问题报告

**日期**：2026-10-18  
**设备**：Netgear Nighthawk X10 R9000（Annapurna Alpine，armv7l）  
**固件**：ImmortalWrt SNAPSHOT r41408-1e93f17910（Linux 6.18.52）  
**测试方式**：固定客户端 + 单变量换线法（a1/a3 对调）  
**结论**：5G 射频链 3/4 的板端 RF 通路异常，天线/线缆正常；当前 5G 可用 2×2（链1/2）

---

## 1. 问题描述

- **现象**：5G 速率偏低（rx ≈115 Mbps、tx ≈60 Mbps），四条射频链 RSSI 中链 3/4 明显偏低（-89 ~ -96 dBm），链 1/2 正常（-52 ~ -70 dBm）。
- **影响**：无法 4×4 MIMO，实际只能 2×2；远距离/高负载下 5G 吞吐受限。
- **复现性**：100%，多次测量一致。
- **关联现象**：与分支 README 中“5G 链路 3/4 RSSI 异常（天线/校准待排查）”一致。

## 2. 测试环境

| 项目 | 值 |
|---|---|
| 设备 | Netgear R9000（Annapurna Labs Alpine，armv7l） |
| 内存 | 1 GB（实测 1,025,068 kB） |
| 固件 | ImmortalWrt SNAPSHOT r41408-1e93f17910 |
| 内核 | 6.18.52 |
| 5G 射频 | QCA9984（ath10k），hw1.0，firmware 10.4b-ct-9984-fW-14 |
| 5G 接口 | 5GHz-radio1，信道 149（5745 MHz），40 MHz，txpower 30 dBm |
| 2.4G | 2.4GHz-radio2，信道 1（2412 MHz），正常 |
| 客户端 | 手机 52:80:fe:dd:4f:78（2×2，NSS2） |
| 环境噪声 | -111 ~ -114 dBm（干净，无外部干扰） |

## 3. 测试方法

固定手机位置，分三阶段测量 5G 四条射频链：

1. **基线**：原接线（a1、a2、a3、a4 各就位）
2. **a3→a1**：a3 线接到 a1 口（a1 线暂空）
3. **完成对调**：a1 线接到 a3 口

每阶段采集：

```sh
iw dev 5GHz-radio1 station dump   # 每链 RSSI / 速率
iw dev 5GHz-radio1 survey dump    # 噪声
```

## 4. 数据记录

### 4.1 每链 RSSI（dBm）

| 链 | 基线 | a3→a1 | 完成对调 |
|---|---|---|---|
| 链1 | -70（a1 线） | -75（a3 线） | **-66（a3 线）** |
| 链2 | -52 | -45 | -78（未动，位置波动） |
| 链3 | -95（a3 线） | -95（空） | **-96（a1 线）** |
| 链4 | -89 | -89 | -91（未动） |

### 4.2 链路指标

| 阶段 | 信号 | 速率 tx / rx | 噪声 |
|---|---|---|---|
| 基线 | -52 dBm | 60.0 MCS3 NSS1 / 115.6 MCS5 NSS2 | -111 dBm |
| a3→a1 | -45 dBm | 26.0 MCS1 NSS2 / 57.8 MCS3 NSS2 | -114 dBm |
| 完成对调 | -66 ~ -72 dBm | 180.0 MCS8 NSS1 / 216~300 MCS5-7 NSS2 | -113 dBm |

## 5. 分析

1. **a3 线正常**：原位（链3）读数 -95；换到链1 后读数 -66 ~ -75，可跑 180 Mbps → 线缆/天线无问题。
2. **链3 异常跟随端口**：a1 线原位（链1）读数 -70（已知良好）；换到链3 后读数 -96 → 异常跟随端口/射频链。后经原厂固件验证（4×4 正常、5G 1733 Mbps），判定为 **ath10k 驱动/校准问题**，而非硬件损坏。
3. **链4 同样异常**：全程 -89 ~ -91，未参与换线，属于同类的射频链问题。
4. **排除干扰**：噪声 -111 ~ -114 dBm，信道 149 干净。
5. **样本限制**：客户端为 2×2，每链 RSSI 来自手机帧；已用换线控制变量，结论成立。最终硬件定性建议用 4×4 客户端或频谱仪复核。

## 6. 结论

- **根因（2026-10-18 修正）**：链3/4 的 -95 dBm 属 **ath10k 驱动/校准层面的异常**，不是板端硬件损坏——原厂固件 U-Boot 明确 `1st Radio 4x4; 2nd Radio 4x4`，且原厂 5G 实测 1733 Mbps（4×4/80MHz），证明四条射频链硬件完好。
- **现状**：5G 可用（链1/2，2×2），实测 rx 216~300 Mbps / tx 180 Mbps。
- **影响**：无法 4×4 MIMO；链4 偏弱可能拉低 MIMO 效率。

## 7. 建议

1. **驱动/校准（优先）**：对比 ath10k `board-2.bin`（QCA9984 hw1.0）与 chainmask 配置；原厂使用 QCA NSS 专有驱动，硬件 4×4 正常，硬件拆机检查可暂缓。
2. **软件**：将 5G chainmask 固定为链1+2（2 条），避免失效链空耗/误判。
3. **文档**：README 的 5G 状态更新为“已启用，2×2 可用，链3/4 硬件异常”。
4. **复测**：以 4×4 客户端或频谱仪复核链3/4 的硬件结论。

## 附录：采集命令

```sh
# 无线接口
iw dev | grep -E "phy#|Interface|ssid|channel"

# 5G 客户端与每链 RSSI
iw dev 5GHz-radio1 station dump | grep -E "Station|signal|bitrate|connected"

# 噪声
iw dev 5GHz-radio1 survey dump | grep -A2 "5745 MHz \[in use\]"

# 射频固件信息
cat /sys/kernel/debug/ieee80211/phy1/ath10k/firmware_info
```

---

## 最终处置（2026-10-18）

- **5G 默认关闭**：`target/linux/alpine/base-files/etc/uci-defaults/99-wifi-preset` 的 `5g` 分支已改为 `disabled='1'`，对应接口（`default_radio1`）一并关闭。
- 保留 `channel=149`、`htmode=VHT80` 参数，供手动启用。
- 手动启用方法：

  ```sh
  uci set wireless.radio1.disabled='0'
  uci set wireless.default_radio1.disabled='0'
  uci commit wireless
  wifi reload
  ```

- 关闭依据：链0/链1 RSSI 相差约 **22 dB**（-75 vs -53 dBm），下行 2×2 空分复用无法建立，速率被压到 **MCS1/NSS1（65 Mbps PHY）**，重传率约 **158%**，实测下行约 **36 Mbps**、上行约 124 Mbps，偶发断线。
- 原厂 QCA NSS 驱动可跑 1733 Mbps（4×4），说明射频硬件正常，属 ath10k 驱动/校准层差异。

---

## 2026-10-19 更正：根因是发射功率过高

此前“5G 不可用 / 链0 异常”的结论不成立。

- **30 dBm（1 W）**：功放进入非线性区，EVM 恶化 → 下行 2 流无法解码 → 退化为 MCS1/NSS1（下行约 36 Mbps，重传约 158%）
- **11 dBm（12 mW）**：链路约 580 Mbps
- **15 dBm（31 mW）**：链路 **650 Mbps（VHT-MCS7 2SS 80MHz）**，跑满宽带
- 同时把信道改为 **36（5180 MHz）**、带宽 80 MHz

结论：5G 可用。默认改为 `channel=36` / `VHT80` / `txpower=15`；保持 ≤ 17 dBm，不要回到 30 dBm。
