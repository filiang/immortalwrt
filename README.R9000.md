# Netgear R9000 · immortalwrt port

本分支基于 [immortalwrt](https://github.com/immortalwrt/immortalwrt) `master`，为 **Netgear Nighthawk X10 (R9000)** 提供 Alpine 平台移植。

## 硬件

| 项目 | 规格 |
|---|---|
| SoC | Annapurna Labs Alpine AL-514（4× Cortex-A15） |
| 内存 | 1 GB DDR3 |
| Flash | 512 MB SLC NAND（Spansion S34MS04G2，2 KB page / 128 KB erase） |
| 2.4G / 5G | 2× QCA9984（ath10k） |
| 60GHz | QCA9500（wil6210，默认禁用） |
| 交换 | RTL8366 / ar8327 系 |

## 本分支改动

- `target/linux/alpine/` — R9000 板级支持、DTS、镜像规则（内核 6.18）
- `target/linux/generic/files/drivers/net/phy/ar8327.{c,h}` — 交换机 port5 状态保留等修复
- `feeds.conf.default` — 新增 alpinefancontrol feed
- `package/kernel/linux/modules/netfilter.mk` — FTP NAT helper
- `package/utils/busybox/files/cron` — cron 支持
- `package/fantastic-packages/`、`package/luci-theme-argon/`、`package/luci-app-argon-config/` — LuCI 主题与附加包

## 构建

```sh
./scripts/feeds update -a && ./scripts/feeds install -a
make menuconfig      # Target: Annapurna Labs Alpine → Netgear R9000
make -j$(nproc) V=s
```

产物：`bin/targets/alpine/generic/immortalwrt-alpine-generic-netgear_r9000-squashfs-factory.img`

## 已知问题

- 5G 链路 3/4 RSSI 异常（天线/校准待排查）
- 蓝牙无驱动（系统未编译 BT 支持）
- 60GHz 默认关闭

## 致谢

- [immortalwrt](https://github.com/immortalwrt/immortalwrt) / [OpenWrt](https://github.com/openwrt/openwrt)
- masmbit/Netgear-R9000-Build、egonenar/openwrt 的参考实现

## 许可

GPL-2.0（与上游一致）
