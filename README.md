# ThinkPad T14 Gen 1 Intel · macOS Sequoia

更新日期：2026-11-19。适用于 **ThinkPad T14 Gen 1 Intel（i7-10510U）**，基于已运行 **macOS Sequoia 15.8.3（24H241）** 的 **OpenCore 1.0.8 RELEASE** 配置整理。

本仓库发布版仅包含 OpenCore，不包含 Ubuntu 引导。SMBIOS 的序列号、MLB、SystemUUID 已清空，ROM 已置零；**使用前必须填写自己的身份信息**。这不是可以不作配置就直接覆盖的完整机器备份。

发布版以 2026-11-19 的内置 EFI 备份为基础，进行了以下整理：

- 替换旧 Monterey EFI，保留键盘修复后的配置和驱动。
- 删除 Ubuntu shim、GRUB 及原 Ubuntu 备用加载器，使用同版本 OpenCore 的 `EFI/BOOT/BOOTx64.efi`。
- 清空 Ubuntu 的 `Misc/BlessOverride`，清除 SMBIOS 机器身份值。
- 删除 Finder 元数据和 `FSCK0000.000` 修复残留。
- 保留驱动包内未启用插件及诊断工具；是否加载以 `config.plist` 为准。

下方硬件测试来自脱敏前的本机运行配置。发布版已通过 ocvalidate 1.0.8 和文件校验，但尚未以填写新身份后的发布版进行整套启动复测。

## 硬件

| 项目 | 实际配置 |
|---|---|
| 机型 | Lenovo ThinkPad T14 Gen 1，20S1A1J7CD，Intel 版 |
| BIOS | N2XET46W 1.36（2026-06-02） |
| CPU | Intel Core i7-10510U，4 核 8 线程 |
| 内存 | 16 GB DDR4，实际 2667 MT/s |
| SSD | WD PC SN730 512 GB NVMe |
| 核显 | Intel UHD，PCI 8086:9b41；macOS 显示 UHD Graphics 620 |
| 独显 | NVIDIA MX330；macOS 下禁用 |
| 屏幕 | LG LP140WFH-SPD1，1920×1080 |
| 声卡 | Realtek ALC257，AppleALC layout-id 86 |
| 无线 / 蓝牙 | Intel AX201 / USB 8087:0026 |
| 有线网络 | Intel I219-V |
| 键盘 / 指点杆 | PS/2 键盘 / Elan TrackPoint |
| 触控板 | Synaptics TM3471-020，SMBus 地址 0x2c |
| 摄像头 | Luxvisions USB UVC，30c9:003b |
| 读卡器 | Realtek RTS522A，未配置专用驱动 |
| SMBIOS | MacBookPro16,2 |

## 功能实测

| 功能 | 结果与边界 |
|---|---|
| 内置启动 | 原机运行配置已通过，OpenCore 为固件第一启动项；发布版需填写身份后自行验证 |
| 核显加速 | Metal 3 已识别，内屏 1080p；显示共享显存 3072 MB |
| 键盘 | 用户确认可正常打字 |
| 触控板 | 双指滚动、轻点正常；轻点需在系统设置中启用 |
| 扬声器 | 用户确认有声音 |
| Wi-Fi | itlwm + HeliPort，用户已连接成功 |
| 有线网络 | 已用于 SSH 与安装后调试 |
| 蓝牙 | 控制器开启，可扫描；无外设，配对及音频尚未测试 |
| 合盖睡眠 / 开盖唤醒 | 一轮短时测试通过，醒后屏幕、键盘、Wi-Fi、触控板及声音均正常 |
| 电池 / CPU 电源管理 | 电池信息可读；X86PlatformPlugin 活跃，xcpm.mode=1；未做续航测量 |
| 未验证 | 摄像头、麦克风录音、耳机口、TrackPoint、外接屏、各 USB 端口、读卡器、长时间待机、深度休眠及 Apple 账户连续互通功能 |

Wi-Fi 使用 HeliPort 1.5.0（应用需单独安装，不在 EFI 内）。联网正常不代表 AirDrop、接力等功能已经验证。SIP 已实测恢复启用；没有针对本机验证 Tahoe 升级。

## BIOS 设置：本机记录

开机按 Enter 后 F1 进入 BIOS；F12 打开启动菜单。下面依据 2026-11-18 的 thinklmi 读取、设置和诊断记录整理，不是本次重新读取的 BIOS 快照。菜单名称和位置以固件实际显示为准。不要先恢复默认值再盲目套用其他机器的清单。

### 已记录为开启

| 设置 | 值 | 说明 |
|---|---|---|
| Intel Hyper-Threading Technology | Enabled | 超线程 |
| Intel Virtualization Technology / VT-x | Enabled | 曾误关；已改回，并在后续诊断中确认 |
| Execute Disable Bit / DEP | Enabled | 执行保护 |
| F12 Boot Menu | Enabled | 固件启动选择菜单 |
| Boot hotkeys / 启动热键 | Enabled | 启动时热键 |
| Ethernet LAN | Enabled | 有线网络 |
| Wireless LAN | Enabled | AX201 Wi-Fi |
| Bluetooth | Enabled | 蓝牙 |
| USB | Enabled | USB 接口 |
| Integrated Camera | Enabled | 摄像头；启用不代表已完成 macOS 实测 |
| Audio | Enabled | 板载声音 |

### 已记录为关闭

| 设置 | 值 | 说明 |
|---|---|---|
| Secure Boot | Disabled | 固件安全启动关闭；与 macOS SIP 是两件事 |
| Security Chip | Disabled | 2026-11-18 已读取确认 |
| Intel TXT / TXTFeature | Disabled | 原为 Enabled，已关闭；后续诊断确认 |
| Intel VT-d Feature | Disabled | 与 VT-x 区分，VT-x 保持开启 |
| Kernel DMA Protection | Disabled | 已记录关闭 |
| Intel SGX | Disabled | 已记录关闭 |
| Thunderbolt BIOS Assist Mode | Disabled | 仅指 BIOS Assist，不等于整个雷雳设备关闭 |
| Wireless WAN | Disabled | WWAN，不是 Wi-Fi |
| Boot Order Lock | Disabled | 允许调整启动顺序 |
| Device Guard | Disabled | 已记录关闭 |

### 数值、锁定状态与未核实项

| 设置 | 状态 | 说明 |
|---|---|---|
| 集显预分配内存 | 256 MB | 本机读取记录；不是 macOS 显示的 3072 MB 共享显存 |
| CFG Lock | **仍锁定** | MSR 0xE2=0x1e008008，bit 15=1；通过 AppleXcpmCfgLock=true 处理，没有解锁 BIOS |
| 启动方式 | 实际为 UEFI/GPT | 已验证 UEFI 启动；没有记录对应 BIOS 菜单的逐项值 |
| 启动顺序 | 本机已将 OpenCore 设为第一启动项 | 固件启动项不随 EFI 文件复制；本仓库不包含 Ubuntu 引导 |
| AMT | 未确认最终值 | 早期仅提出可选禁用，不写成已经关闭 |
| Thunderbolt Security Level / 整体设备开关 | 未重新确认 | 旧 Monterey README 的 NO SECURITY / DISABLED 不作为当前证据 |
| CSM、Above 4G Decoding、XHCI Hand-off | thinklmi 未提供对应项 | 未声称已开启或关闭，不要求修改隐藏设置 |
| Fast Boot、SATA 模式、OS Type、Legacy RTC、串口/并口 | 无本次实测记录 | 本机系统盘是 NVMe；删除旧 README 的泛用设置要求 |

## EFI 内容与启动方式

```text
EFI/
├── BOOT/       # OpenCore 1.0.8 RELEASE 标准备用入口
└── OC/         # 当前 OpenCore 及配置
EFI-SHA256.txt  # 仓库内 EFI 文件校验清单
```

`EFI/BOOT/BOOTx64.efi` 是 OpenCore 1.0.8 RELEASE 的标准备用启动入口；固件也可直接登记 `\EFI\OC\OpenCore.efi`。仓库中没有 Ubuntu shim、GRUB 或 Ubuntu BlessOverride。

### 使用前配置

1. 备份目标机器当前 EFI 和重要数据，确认目标 ESP；不要格式化整盘。
2. 为 `PlatformInfo/Generic` 配置自己的 `SystemSerialNumber`、`MLB`、`SystemUUID` 和六字节 `ROM`，机型为 `MacBookPro16,2`。
3. 对照本机硬件检查 USB 映射、核显、声卡及 ACPI 配置。本仓库并非 AMD 版 T14 的配置。
4. 单独安装 HeliPort；本仓库提供 itlwm 驱动，不包含 HeliPort 应用。
5. 先通过可恢复的启动介质测试，确认可启动后再部署到内置 ESP，并配置固件默认启动项。

本次仓库整理没有修改原机正在使用的 EFI，也没有删除原机的 Ubuntu 分区。仓库当前版本已脱敏，旧 Git 提交历史不在本次脱敏范围内。

## 关键配置

- `SetupVirtualMap=true`：本机启动对照验证必需；false 曾卡在 EXITBS:START。
- `EnableWriteUnprotector=false`、`RebuildAppleMemoryMap=true`、`SyncRuntimePermissions=true`。
- `AppleXcpmCfgLock=true`；`DisableIoMapper=true`；`XhciPortLimit=false`，使用现有 USB 映射。
- 核显伪装 device-id 9b3e，AAPL,ig-platform-id 字节为 `00009B3E`；声卡 layout-id 86。
- ACPI：SSDT-PLUG、EC-USBX、AWAC-DISABLE、PNLF、GPRW、dGPU-Off；独显路径 RP09.PXSX。
- 保留诊断启动参数：`-v keepsyms=1 debug=0x100 igfxonln=1 -igfxblr -wegnoegpu`。启动时有文字是当前配置的正常行为。
- RELEASE 引导；Debug Target=0；内建选择器显示，超时 8 秒；LauncherOption=Disabled。
- 固件 Secure Boot 关闭，OpenCore SecureBootModel=Disabled；macOS SIP 实测 enabled。
- 启用 PS2Keyboard、PS2Mouse、PS2Trackpad；触控板通过 PS2 初始化后转交 RMISMBus。仅启用 RMI 包内 VoodooInput，禁用 PS2 包内同名插件，避免重复加载。

### 主要驱动版本

| 驱动 | 版本 |
|---|---|
| Lilu / WhateverGreen / AppleALC | 1.7.2 / 1.7.1 / 1.9.8 |
| VirtualSMC / SMCProcessor / SMCBatteryManager | 1.3.8 |
| ECEnabler / NVMeFix / IntelMausi | 1.0.6 / 1.1.3 / 1.0.8 |
| USBToolBox / UTBMap | 1.2.0 / 本机映射，包版本 1.1 |
| VoodooPS2Controller | 2.3.7 |
| VoodooRMI / VoodooSMBus / VoodooInput | 1.4.3 / 3.0 / 1.1.7 |
| itlwm | 2.3.0，搭配 HeliPort 1.5.0 |
| IntelBluetoothFirmware / IntelBTPatcher | 2.4.0 |
| BlueToolFixup | 2.7.2 |
| BrightnessKeys / RestrictEvents | 1.0.3 / 1.1.6 |

## 已解决的问题

1. **EXITBS:START 卡住**：SetupVirtualMap 改为 true 后通过。
2. **安装 U 盘 Still waiting for root device**：安装卷内容是 HFS+，但 GPT 类型仍是 Microsoft basic data；修正为 Apple HFS/HFS+ 后进入安装界面，换 USB 口本身没有解决。
3. **安装器损坏**：日志报 `pkgdmg is missing a footer`。安装应用的 SharedSupport.dmg 错用了解包出的纯 DMG，应保留完整 InstallAssistant.pkg 的 pkgdmg 封装；恢复正确文件并校验后安装成功。macOS 目标卷使用 APFS。
4. **内置启动后键盘失效**：EFI 中键盘 kext 目录缺失，出现 FSCK0000.000；恢复键盘驱动、校验和检查 FAT 后重启，驱动加载且用户确认可输入。目录异常的起因未确定。
5. **F12 不显示 OpenCore**：仅 create-only 登记并设置 BootNext 不够；将 OpenCore 加入 BootOrder 后解决，最终按用户要求设为第一启动项。

## 校验与恢复

下载后、填写身份值前，可在仓库根目录执行 `shasum -a 256 -c EFI-SHA256.txt` 检查发布文件。填写身份或修改配置后，config.plist 的校验值会变化，这是预期现象；应另行使用同版本 ocvalidate 检查配置结构。静态校验通过不等于硬件功能已验证。

本仓库只包含引导配置，不包含 macOS 安装器、操作系统或用户数据，也不保存固件启动顺序。

恢复时优先使用自己的完整 EFI 备份。若使用本仓库发布版，须先填写有效的 SMBIOS 身份信息，再部署到目标 ESP，并检查固件中的 OpenCore 路径。本仓库不提供 Ubuntu 引导；如果目标机仍需双系统，应自行保留其 EFI/ubuntu。不要仅拷贝文件就假设固件启动顺序会自动恢复。修改完成后同步并安全卸载 ESP。

## 项目来源

- [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
- [VoodooRMI 安装说明](https://github.com/VoodooSMBus/VoodooRMI#installation)
- [itlwm 与 HeliPort](https://github.com/OpenIntelWireless/itlwm)
- [IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware)
- [BlueToolFixup / BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM)

---

Powered by **OpenAI Codex**.
