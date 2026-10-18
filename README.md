# ImmortalWrt for Netgear R9000（Annapurna Alpine 平台）

本分支 `r9000-alpine-port` 基于 **ImmortalWrt master** 进行第三方设备层重构与移植，整合了社区中针对 Alpine 架构（Annapurna Labs AL-314）及 `ar8327` 交换芯片的修复补丁，可编译出内核 6.18 的 ImmortalWrt 固件。

## 硬件

| 项目 | 规格 |
|---|---|
| SoC | Annapurna Labs Alpine **AL-314**（4× Cortex-A15 @1.7GHz；原厂启动日志：Device ID = a314 / AL31400-1700） |
| 内存 | 1 GB DDR3 |
| Flash | 512 MB SLC NAND（Spansion S34MS04G2，2 KB page / 128 KB erase） |
| 2.4G / 5G | 2× QCA9984（ath10k） |
| 60GHz | QCA9500（wil6210，默认禁用） |
| 交换 | ar8327 系 |

## 🌐 默认网络参数

| 项目 | 默认值 | 说明 |
|---|---|---|
| **LAN IP** | **`10.0.0.1/24`** | 首启由 `target/linux/alpine/base-files/etc/uci-defaults/99-lan-ip` 设置，避免与国内光猫常见的 `192.168.1.1` 网段冲突导致进不了管理页 |
| Web 管理 | `http://10.0.0.1` | LuCI；**默认 root 无密码**，首次登录请立即设置 |
| SSH | `ssh root@10.0.0.1` | 同样默认无密码，建议配置公钥后关闭密码登录 |
| DHCP | `10.0.0.100` 起 | 由 dnsmasq 自动生成 |
| 2.4G Wi-Fi | 信道 1 / HT40，SSID `Openwrt`，密码 `12345678` | 由 `99-wifi-preset` 预设，请自行修改 |
| 5G Wi-Fi | 信道 **149** / VHT80，txpower **15 dBm（固定）**，SSID `Openwrt`，密码 `12345678` | 30 dBm 功放过载、16 dBm 实测掉线；**仅用 15 dBm，实测稳定跑满宽带** |
| 60G Wi-Fi | 默认关闭；接口 `60GHz-radio0`，SSID `R9000-60G` | 默认禁用，**尚未实测**，稳定性未知 |
| 监管域 | `CN` | 5G 可用信道按中国区 |

## 🛠️ 分支目前状态

* **[已修复] 网口逻辑顺序**：修正上游源码中 WAN/LAN 错乱的问题，有线网络、拨号及有线转发完全正常。
* **[已修复] 2.4G Wi-Fi**：驱动加载正常，通过高负载长时测试，运行稳定。
* **[可用] 5G Wi-Fi（2×2 / 15 dBm 固定）**：`channel 149 / VHT80`，`chainmask=0x3` 固定 2×2，`txpower=15 dBm`。实测稳定，下行 260–520 Mbps、上行 433–650 Mbps，跑满宽带。此前“不可用”是默认 **30 dBm** 功放非线性导致下行 MCS1；实测 **16 dBm 会让设备掉线**，因此只使用 15 dBm，不再调整功率。
* **[默认关闭，未实测] 60GHz Wi-Fi (802.11ad)**：编译配置中默认关闭，**未做稳定性或吞吐测试**，不做可用性判断；接口名 `60GHz-radio0`，SSID 预设为 `R9000-60G`（不再继承默认名 `ImmortalWrt`）。如需启用请自行评估。
* **[已屏蔽] 10G SFP+ 光口**：为提升整体内核及网络稳定性，默认不启用。

> 💡 **推荐定位**：本固件适合将 R9000 作为 **有线高性能主路由/弱电箱拨号网关**，或 **2.4G + 5G（80MHz）智能家居 AP** 使用。5G 请保持 txpower ≤ 17 dBm；有 4×4/更高无线需求者建议通过千兆 LAN 口外接纯 AP。

## 📦 已验证构建

| 项目 | 值 |
|---|---|
| 内核 | 6.18.52（`target/linux/alpine`，25 个内核补丁 + 1 个 ath10k backports 补丁） |
| 目标 | `alpine/generic` → `DEVICE_netgear_r9000` |
| 最近重建 | 2026-10-18 19:55 GMT+8，0 错误 |
| factory.img | 61,866,113 字节 |
| sha256 | `74f01683725b87c6a42bac65661c12bc6afb88fec3f0ebb07cc9b5376d4a5782` |
| manifest | 369 个包 / 10 个 zh-cn 语言包 |

## 📋 实测日志（脱敏）

`docs/logs/` 收录本机实测串口与刷机日志，已脱敏（MAC 仅保留 OUI、后三字节替换为 `xx`；SSID `NETGEAR09*` 替换为 `MYSSID*`）：

| 文件 | 内容 |
|---|---|
| `docs/logs/boot-stock-nmrp.log` | 原厂固件 + nmrpflash 刷机全过程 |
| `docs/logs/boot-ath10k-ct.log` | ath10k-ct 版本启动日志 |
| `docs/logs/boot-ath10k.log` | 上游 ath10k 版本启动日志 |
| `docs/logs/boot-2x2.log` | chainmask 0x3（2×2）版本启动日志 |

## 🚀 快速编译

1. 克隆本分支：

```bash
git clone -b r9000-alpine-port --single-branch https://github.com/filiang/immortalwrt.git
cd immortalwrt
```

2. 更新并安装 feeds（**必须**，否则 LuCI/插件选项会在 oldconfig 时被丢弃）：

```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

3. 载入本仓库附带的已验证配置（推荐）：

```bash
cp r9000.config .config
```

或手动 `make menuconfig`：

* **Target System**: `Annapurna Labs Alpine`
* **Target Profile**: `Netgear R9000`
* 中文界面：`LuCI → Translations → Chinese (Simplified)`

4. 开始编译（容器/root 环境需 `FORCE_UNSAFE_CONFIGURE=1`）：

```bash
FORCE_UNSAFE_CONFIGURE=1 make -j$(nproc) V=s
```

5. 产物：

```
bin/targets/alpine/generic/immortalwrt-alpine-generic-netgear_r9000-squashfs-factory.img
bin/targets/alpine/generic/immortalwrt-alpine-generic-netgear_r9000-squashfs-sysupgrade.bin
```

> ⚠️ 请勿在线使用 `opkg` 单独升级内核模块（kmod）；如有插件需求，请在 menuconfig 中勾选并随固件整体编译。

## 💾 刷机与救砖

### 首次从原厂/其他系统刷入（TFTP）

必须在 **TFTP 恢复模式** 下写入 `*-factory.img`：

1. 拔掉路由器电源。
2. 用顶针按住背面 `Reset` 键不放。
3. 插上电源，持续按住 `Reset`。
4. 观察电源指示灯（Power LED）：闪烁橙色 → 缓慢闪烁白色 → 第一波快速闪烁白色 → 极快闪烁白色。
5. 在指示灯**极快闪烁白色**时释放 `Reset`，进入标准 TFTP 接收状态。
6. 用 Linux `tftp-hpa` 或 Windows `Tftpd64`，将 `*-factory.img` 推送到 `192.168.1.1`。
7. **推送完成后静置路由器 5 分钟以上**，等待解压、写入并自动重启。

### 后续升级

* 在系统内通过 LuCI 后台上传 `*-sysupgrade.bin`。
* **强烈建议**：跨版本升级时**取消勾选"保留配置"**，干净刷入（Clean Flash），避免魔改后的网口命名冲突。

## 🙏 致谢

* [ImmortalWrt](https://github.com/immortalwrt/immortalwrt) / [OpenWrt](https://github.com/openwrt/openwrt)
* masmbit/Netgear-R9000-Build、egonenar/openwrt 的参考实现
* fantastic-packages、luci-theme-argon 等社区包

## ⚠️ 免责声明

本分支属于个人技术兴趣维护，底层涉及非官方支持的硬改与屏蔽，刷机存在风险。作者不对因刷写本固件导致的任何形式的硬件损坏（砖机）、数据丢失或网络故障承担责任。请在充分知晓 TFTP 救砖方法的前提下理性折腾。

## 📄 许可

GPL-2.0（与上游一致）
