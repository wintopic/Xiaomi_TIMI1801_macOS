# 小米游戏本八代 TIMI1801 OpenCore 配置

本目录保存小米游戏本八代 TIMI1801 的一份实机可用 OpenCore EFI。目录名 `WDS500G1X0E-00AFY0` 来自当前验证机器的 WD 系统盘型号，用于区分不同硬盘/配置版本。

当前 EFI 已验证可完成 macOS 启动、核显加速、内屏显示、Type-C 外接显示、内置/耳机音频、Wi-Fi/蓝牙/有线网卡配置加载，以及 BetterDisplay 下的外接屏 HiDPI 显示。

## 目录结构

```text
WDS500G1X0E-00AFY0/
├── EFI/
│   ├── BOOT/
│   └── OC/
│       ├── ACPI/
│       ├── Drivers/
│       ├── Kexts/
│       ├── Resources/
│       ├── Tools/
│       ├── OpenCore.efi
│       └── config.plist
├── config.plist
├── Diagnostics/
│   ├── SPDisplaysDataType.txt
│   ├── SPAudioDataType.txt
│   ├── HDEF-音频.txt
│   ├── IGPU-显示.txt
│   ├── CoreAudio设备.json
│   └── nvram-关键项.txt
├── Skill/
│   └── xiaomi-gaming-8th-hackintosh/
└── README.md
```

`EFI/` 是完整 OpenCore 配置；`config.plist` 是单独副本；`Diagnostics/` 是当前可用状态的实机检测结果；`Skill/` 是面向 Codex 的安装维护说明。

## 硬件信息

| 类别 | 型号/状态 |
| --- | --- |
| 机型 | 小米游戏本八代 TIMI1801 |
| SMBIOS | `MacBookPro15,3` |
| CPU | Intel Core i7 六核，i7-8750H 同级 |
| 内存 | 16 GB |
| 核显 | Intel UHD Graphics 630，Device ID `0x3e9b` |
| 独显 | 已通过 ACPI 禁用 |
| 内屏 | `LM156LF9L01`，1920 x 1080 @ 60Hz |
| 外接屏 | AOC 24G1WG4，EDID 名称 `24V1W` |
| Type-C 转接器 | UGREEN CM136-70495 |
| 声卡控制器 | Intel Cannon Lake-H/S cAVS `8086:a348` |
| 音频 Codec | Realtek ALC1220 |
| 显示器音频 Codec | Intel Display Audio `8086:280b`，尚未稳定启用 |
| 有线网卡 | RealtekRTL8111 |
| Wi-Fi | Intel Wi-Fi，`itlwm.kext` |
| 蓝牙 | Intel Bluetooth |
| USB | USBToolBox + UTBMap |
| 来源系统盘 | WD `WDS500G1X0E-00AFY0` |

## 功能状态

| 功能 | 状态 | 说明 |
| --- | --- | --- |
| OpenCore 启动 | 可用 | `MacBookPro15,3` |
| macOS 15 | 可用 | 验证系统版本为 macOS 15.7.7 |
| 核显加速 | 可用 | Intel UHD Graphics 630，Metal 正常 |
| 内屏 | 可用 | 1920 x 1080 @ 60Hz |
| Type-C 外接显示 | 可用 | UGREEN CM136 转 AOC 24G1WG4 |
| 外接屏 HiDPI | 可用 | BetterDisplay 下显示 `2880 x 1620`，界面看起来像 `1440 x 810` |
| 内置/耳机音频 | 可用 | AppleALC ALC1220 layout 98 |
| Wi-Fi | 配置已包含 | `itlwm.kext`，需填写自己的 Wi-Fi 信息或使用 HeliPort |
| 蓝牙 | 配置已包含 | IntelBluetoothFirmware + IntelBTPatcher + BlueToolFixup |
| 有线网卡 | 配置已包含 | RealtekRTL8111 |
| USB 映射 | 配置已包含 | USBToolBox + UTBMap |
| 亮度快捷键 | 配置已包含 | BrightnessKeys |
| 读卡器 | 配置已包含 | Sinetek-rtsx |
| 独显 | 已禁用 | `SSDT-Disable_GPU_PEG0.aml` |
| HDMI/DP 显示器音频 | 未稳定 | AOC 耳机口音频在 macOS 中尚未作为输出设备出现 |

## 安装前准备

建议准备：

- macOS 安装 U 盘
- ProperTree、OpenCore Configurator 或其他 plist 编辑器
- GenSMBIOS 或等效 SMBIOS 生成工具
- BetterDisplay，用于 1080p 外接屏 HiDPI 显示
- 一个可用于恢复 EFI 的备用 U 盘

BIOS 建议：

- 关闭 Secure Boot
- 使用 AHCI 模式
- 优先从 OpenCore 启动项启动
- 如果 BIOS 可关闭 CFG Lock，建议关闭；如果没有选项，保留当前 OpenCore Quirk 配置

## 首次配置

公开配置中的 SMBIOS 信息为占位符。安装前打开 `EFI/OC/config.plist`，在 `PlatformInfo -> Generic` 中生成并填写自己的值：

```text
SystemProductName = MacBookPro15,3
SystemSerialNumber = CHANGEME000000
MLB = CHANGEME0000000000000
SystemUUID = 00000000-0000-0000-0000-000000000000
ROM = <000000000000>
```

如果使用内置的 `itlwm.kext` 自动连接 Wi-Fi，还需要修改：

```text
EFI/OC/Kexts/itlwm.kext/Contents/Info.plist
WiFiConfig -> WiFi_1 -> ssid = CHANGE_ME_SSID
WiFiConfig -> WiFi_1 -> password = CHANGE_ME_PASSWORD
```

也可以保持占位符不变，进入系统后使用 HeliPort 手动连接无线网络。

## 安装方式

1. 制作 macOS 安装 U 盘。
2. 挂载安装 U 盘的 EFI 分区。
3. 将本目录的 `EFI/` 复制到 U 盘 EFI 分区根目录。
4. 按“首次配置”填写 SMBIOS 和 Wi-Fi 信息。
5. 检查 `EFI/OC/config.plist`：

```zsh
plutil -lint EFI/OC/config.plist
```

6. 从 U 盘 OpenCore 启动 macOS 安装器。
7. 完成 macOS 安装并进入系统。
8. 挂载系统盘 EFI 分区，将同一份 `EFI/` 复制到系统盘 EFI。
9. 安装 BetterDisplay，并设置外接屏 HiDPI。
10. 运行“验证命令”确认显示和音频状态。

## 系统盘 EFI 写入

macOS 下磁盘编号会随启动顺序变化，写入前先确认当前系统盘：

```zsh
diskutil info /
diskutil info /dev/<APFS Physical Store>
diskutil info /dev/<whole disk>
```

挂载系统盘 EFI：

```zsh
sudo diskutil mount /dev/<whole disk>s1
```

复制 EFI：

```zsh
ditto EFI /Volumes/EFI/EFI
```

复制完成后卸载：

```zsh
diskutil unmount /Volumes/EFI
```

## OpenCore 组件

Drivers：

- `HfsPlus.efi`
- `OpenRuntime.efi`
- `ResetNvramEntry.efi`

ACPI：

- `SSDT-ALS0.aml`
- `SSDT-SBUS.aml`
- `SSDT-Disable_GPU_PEG0.aml`
- `SSDT-EC.aml`
- `SSDT-GPI0.aml`
- `SSDT-MCHC.aml`
- `SSDT-PMC.aml`
- `SSDT-PLUG.aml`
- `SSDT-PNLF.aml`
- `SSDT-USBX.aml`
- `SSDT-XOSI.aml`

主要 Kext：

- Lilu
- WhateverGreen
- AppleALC
- VirtualSMC
- SMCBatteryManager / SMCLightSensor / SMCProcessor / SMCSuperIO
- NVMeFix
- RestrictEvents
- USBToolBox / UTBMap / XHCI-unsupported
- IntelBluetoothFirmware / IntelBTPatcher / BlueToolFixup
- itlwm
- RealtekRTL8111
- BrightnessKeys
- Sinetek-rtsx
- VoodooPS2Controller

## 核显与 Type-C 外接显示

Type-C 外接显示输出已映射到 IGPU framebuffer con2，对应 macOS 中的 `AppleIntelFramebuffer@2`。已验证 UGREEN CM136 转接器连接 AOC 24G1WG4 可正常亮屏。

IGPU 设备路径：

```text
PciRoot(0x0)/Pci(0x2,0x0)
```

核心平台参数：

```text
AAPL,ig-platform-id = <0900a53e>
framebuffer-patch-enable = <01000000>
framebuffer-portcount = <03000000>
framebuffer-pipecount = <03000000>
framebuffer-stolenmem = <00003001>
framebuffer-fbmem = <00009000>
```

Type-C / AOC 使用 con2：

```text
framebuffer-con2-enable = <01000000>
framebuffer-con2-index = <02000000>
framebuffer-con2-busid = <04000000>
framebuffer-con2-pipe = <0a000000>
framebuffer-con2-type = <00040000>
framebuffer-con2-flags = <87010000>
framebuffer-con2-has-lspcon = <01000000>
framebuffer-con2-preferred-lspcon-mode = <01000000>
enable-lspcon-support = <01000000>
enable-hdmi20 = <01000000>
```

外接显示稳定相关参数：

```text
disable-agdc = <01000000>
force-online = <01000000>
force-online-framebuffers = <0200000000000000>
complete-modeset = <01000000>
complete-modeset-framebuffers = <0200000000000000>
```

启动参数包含：

```text
-igfxblt igfxonln=1 -igfxtypec -igfxlspcon igfxonlnfbs=2 igfxfcms=1 igfxfcmsfbs=2 igfxagdc=0
```

外接 AOC 24G1WG4 使用 `AAPL02,override-no-connect` 注入 256 字节 EDID。

## 外接屏显示效果

AOC 24G1WG4 是 1080p 显示器。推荐使用 BetterDisplay 创建 HiDPI 模式：

```text
实际渲染分辨率: 2880 x 1620
界面显示效果:   1440 x 810 @ 60Hz
```

字体平滑建议：

```zsh
defaults -currentHost write -globalDomain AppleFontSmoothing -int 1
defaults write -globalDomain AppleFontSmoothing -int 1
defaults write -globalDomain CGFontRenderingFontSmoothingDisabled -bool NO
```

颜色配置建议使用系统为 `24V1W` 生成的 ColorSync 配置文件。

## 音频配置

Realtek ALC1220 使用 AppleALC layout 98：

```text
PciRoot(0x0)/Pci(0x1f,0x3)
layout-id = <62000000>
alc-layout-id = <62000000>
```

`0x62` 的十进制是 `98`。如果需要命令写入，使用：

```zsh
plutil -replace 'DeviceProperties.Add.PciRoot(0x0)/Pci(0x1f,0x3).layout-id' -data YgAAAA== EFI/OC/config.plist
plutil -replace 'DeviceProperties.Add.PciRoot(0x0)/Pci(0x1f,0x3).alc-layout-id' -data YgAAAA== EFI/OC/config.plist
```

当前已验证内置/耳机音频可出声。AOC 显示器音频输出尚未稳定出现在 macOS 输出设备列表中。

## 验证命令

显示和音频：

```zsh
system_profiler SPDisplaysDataType SPAudioDataType
```

HDA Codec：

```zsh
ioreg -r -c IOHDACodecDevice -l | rg -n "IOHDACodecDevice|IOHDACodecVendorID|IOHDACodecAddress"
```

音频布局：

```zsh
ioreg -p IOService -n HDEF -r -l | rg -n "layout-id|alc-layout-id|HDAConfigDefault|CodecName|PinConfigurations" -C 2
```

外接显示映射：

```zsh
ioreg -p IOService -n IGPU -r -l | rg -n "framebuffer-con2|disable-agdc|force-online|complete-modeset|AAPL02,override-no-connect|audio-selector" -C 1
```

预期状态：

- Intel UHD Graphics 630 正常加载
- 内屏 `LM156LF9L01` 在线
- 外接屏 `24V1W` 在线
- BetterDisplay 下外接屏为 `2880 x 1620`，界面看起来像 `1440 x 810`
- AppleALC 加载 ALC1220 layout 98
- `Built-in Output` 可播放声音

## 诊断快照

`Diagnostics/` 保存了当前可用配置下的状态快照：

- `SPDisplaysDataType.txt`
- `SPAudioDataType.txt`
- `HDEF-音频.txt`
- `IGPU-显示.txt`
- `CoreAudio设备.json`
- `nvram-关键项.txt`

这些文件用于对照安装后的实际状态。

## Codex 安装维护 Skill

本目录附带一个 Codex Skill：

```text
Skill/xiaomi-gaming-8th-hackintosh/
```

可复制到：

```text
~/.codex/skills/xiaomi-gaming-8th-hackintosh
```

使用示例：

```text
Use $xiaomi-gaming-8th-hackintosh to install or repair the Xiaomi Gaming Laptop 8th-gen Hackintosh EFI.
```

Skill 中记录了当前配置细节、恢复流程、验证命令和已知调试结论。

如果要把这款小米游戏本八代迁移到其他固态、其他网卡、其他外接显示器或不同硬件组合，请优先使用仓库根目录的通用 Skill：

```text
AI通用调试SKILL/xiaomi-gaming-8th-auto-debug/
```

## 致谢

本项目基于 OpenCore 及多个社区驱动项目。相关二进制文件版权与许可证归各自上游项目所有。
