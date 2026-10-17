# Annapurna Alpine Target for ImmortalWrt

## 概述

Netgear R9000 (Annapurna Labs Alpine SoC) 适配 ImmortalWrt 的补丁集合。

## 内核版本

- **目标内核**: 6.18 (ImmortalWrt 标准)
- **原始内核**: 6.6 (来自 masmbit/Netgear-R9000-Build)

## 补丁说明

共 **26 个补丁**，全部已通过 Kernel 6.18 dry-run 验证：

| 范围 | 内容 | 数量 |
|------|------|------|
| 009 | 合并的PCI ID补丁 (原091+121+151) | 1 |
| 010 | R9000设备树 | 1 |
| 020-080 | ARM架构/GIC/GPIO/PCI/EDAC | 12 |
| 090-092 | DMA RAID加速引擎及PCI quirk | 2 |
| 100-110 | MTD NAND / MDIO GPIO | 2 |
| 120-122 | 以太网驱动支持及PCI quirk | 3 |
| 130 | ar8327 port5支持 (已直接写入immortalwrt driver) | 0 (已合并) |
| 140-152 | 温控/加密/sg_split修复 | 3 |
| 160 | g761温控 (已 upstream，已删除) | 0 (已删除) |
| 170 | XR700设备树 | 1 |

## 重基修改记录

### 合并的补丁
- **009**: 合并原 091(DMA PCI ID) + 121(ETH PCI ID) + 151(Crypto PCI ID)，解决同一行冲突
- **160**: 删除，g761已在6.18内核上游支持

### 手动修复的补丁
- **092**: 更新DMA PCI quirk上下文 (quirk位置从3121→3257行)
- **120**: 重写Ethernet Kconfig/Makefile补丁 (自动生成diff)
- **122**: 更新ETH PCI quirk上下文
- **152**: 修复sg_split的两个hunk (行号/结构调整)
- **130**: ar8327 port5支持改为直接修改immortalwrt的generic驱动文件

### 直接修改的文件 (非补丁)
- `target/linux/generic/files/drivers/net/phy/ar8327.h` - 添加port5_status字段
- `target/linux/generic/files/drivers/net/phy/ar8327.c` - 添加port5处理逻辑

## 构建步骤

```bash
# 1. 确保内核补丁目录已重命名为 patches-6.18
# 2. 确保config文件已重命名为 config-6.18
# 3. 运行构建
make defconfig
make menuconfig  # 选择 target: alpine
make V=sc -j$(nproc)
```

## 来源

- 原始补丁: https://github.com/masmbit/Netgear-R9000-Build
- OpenWrt fork: https://github.com/egonenar/openwrt
- 重基目标: https://github.com/immortalwrt/immortalwrt
