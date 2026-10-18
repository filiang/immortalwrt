# R9000 原厂固件启动流程（串口日志整理）

> 来源：`/home/pika/serial-r9000.log`（2026-10-18，492 KB / 9,888 行）
> 原厂固件：Netgear R9000 V1.0.6.46

## 启动阶段（按日志顺序）

### 1. Stage 2（RAM loader）
```
Annapurna Labs stage 2: stage2_eth3_ram_loader v1.65.1
Executing next!
Annapurna Labs stage 2: stage2.5_loader v1.65.1
```

### 2. Stage 3
```
Stage 3 version: 1.65.1
Commit ID: e88c9c4
CVOS commit ID: d32367c
HAL commit ID: 8b8f7b5
Build date: May 20 2015 19:57:35
EEPROM Revision ID = 38
Device ID = a314
Device Info: AL31400-1700
Loading DT to 00100000 (17996 bytes)...
Board config ID: alpine_db (hw29765235p0p512p1024p4x4p4x4)
```

### 3. EEPROM 初始化
```
eeprom_per_device_init: no valid information found!
eeprom_per_device_init#104: Try reading from NAND flash
eeprom_per_device_init#116: Reading from NAND flash OK
```

### 4. U-Boot
```
U-Boot 2015.01-gd836bbb (Jul 22 2016 - 13:32:00)
U-boot dni1 V1.8 for DNI HW ID: 29765235
NOR flash 0MB; NAND flash 512MB; RAM 1024MB; 1st Radio 4x4; 2nd Radio 4x4
I2C: ready
DRAM: 1 GiB
```

### 5. 内核
```
Starting kernel ...
Uncompressing Linux... done, booting the kernel.
Linux version 3.10.20-al-5.0-ga_na (gcc 4.8.3) #1 SMP Mon Aug 18 11:01:24 CST 2025
Machine: AnnapurnaLabs Alpine (Device Tree), model: Annapurna Labs Alpine Dev Board
Kernel command line: root=/dev/mtdblock5 rootfstype=squashfs ubi.mtd=netgear
                    pci=pcie_bus_perf console=ttyS0,115200
```

### 6. 用户态（sysinit）
```
Please press Enter to activate this console.
sysinit: ECC failed: 0
sysinit: ECC corrected: 0
sysinit: Number of bad blocks: 3
sysinit: Block size 131072, page size 2048, OOB size 128
sysinit: Generating Rules... Done!
sysinit: Starting Firewall... Done!
```

## 关键硬件事实

| 项目 | 值 |
|---|---|
| SoC | Device ID = a314 / AL31400-1700（Alpine AL-314） |
| DRAM | 1 GiB |
| NAND | 512 MB（坏块 3 个，ECC 无错误） |
| 射频 | 1st Radio 4x4；2nd Radio 4x4 |
| 原厂内核 | Linux 3.10.20-al-5.0 |

## 非致命报错（原厂自带）

- `UBI error: ubi_read_volume_table: the layout volume was not found`（mtd7/mtd8）
- `sysinit: Invalid command : dni_vlan_pri`
- `qca-nss-ecm.conf` 缺失、`nf_nat_dnisip.ko` 缺失

## 关键结论

1. **四条射频链硬件正常**：U-Boot 明确 `1st Radio 4x4; 2nd Radio 4x4`，原厂 5G 可达 1733 Mbps（4×4/80MHz）
2. 因此我们编译版 5G 链3/4 的 -95 dBm 问题指向 **ath10k 驱动/校准（board-2.bin）/预校准数据**
3. 日志中出现两次完整启动（恢复模式一次 + 正常启动一次），属正常现象
