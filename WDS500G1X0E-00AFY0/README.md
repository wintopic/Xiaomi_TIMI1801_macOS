# 小米游戏本八代黑苹果配置

本目录是小米游戏本八代 TIMI1801 的当前稳定 OpenCore 配置归档，按当前 macOS 系统硬盘型号 `WDS500G1X0E-00AFY0` 命名。

这份配置来自一台已经可正常启动、外接屏可亮、内置音频可出声的机器。公开版已经去除真实 SMBIOS 序列号信息。

## 目录结构

```text
WDS500G1X0E-00AFY0/
├── EFI/                         # 去敏后的完整 OpenCore EFI
├── config.plist                  # 与 EFI/OC/config.plist 相同的单独副本
├── Diagnostics/                  # 备份时的显示、音频、NVRAM、IOReg 快照
├── Skill/xiaomi-gaming-8th-hackintosh/
│   ├── SKILL.md                  # Codex 安装维护 Skill
│   └── references/current-build.md
└── README.md
```

## 机器信息

- 机型：小米游戏本八代 TIMI1801
- SMBIOS：`MacBookPro15,3`
- CPU：6 核 Intel Core i7，i7-8750H 同级
- 核显：Intel UHD Graphics 630，`0x3e9b`
- 声卡：Intel HDA `8086:a348`，Realtek ALC1220
- 内屏：`LM156LF9L01`，1920 x 1080 @ 60Hz
- 已验证外接屏：AOC 24G1WG4，EDID 名称 `24V1W`
- 已验证转接器：绿联 UGREEN CM136-70495 USB-C 多功能转换器
- 当前来源系统盘：WD `WDS500G1X0E-00AFY0`

## 当前可用功能

| 功能 | 状态 | 说明 |
| --- | --- | --- |
| macOS 启动 | 可用 | OpenCore，SMBIOS `MacBookPro15,3` |
| 内屏显示 | 可用 | 1920 x 1080 @ 60Hz |
| 核显加速 | 可用 | Intel UHD Graphics 630，Metal 正常 |
| Type-C 外接显示 | 可用 | UGREEN CM136 到 AOC 24G1WG4，走 IGPU framebuffer con2 |
| 外接屏清晰文字 | 可用 | 建议配合 BetterDisplay HiDPI |
| 内置/耳机音频 | 可用 | ALC1220，AppleALC layout 98 |
| Wi-Fi | 可用配置已包含 | `itlwm.kext` |
| 蓝牙 | 可用配置已包含 | IntelBluetoothFirmware、IntelBTPatcher、BlueToolFixup |
| 有线网卡 | 可用配置已包含 | RealtekRTL8111 |
| USB 映射 | 可用配置已包含 | USBToolBox + UTBMap |
| 独显 | 已禁用 | `SSDT-Disable_GPU_PEG0.aml` |
| 读卡器 | 配置已包含 | Sinetek-rtsx |

## 已知问题

- AOC 显示器的 HDMI/DP 音频尚未作为 macOS 输出设备稳定出现。
- DDC/DDC-CI 通过 Type-C 转 HDMI 转接器不稳定，曾导致系统卡死，不建议使用。
- BetterDisplay 对外接 1080p 显示器的文字清晰度很关键，卸载后可能退回普通 1080p 模式，文字会变虚。
- 公开版 `config.plist` 已去敏，不能不改 SMBIOS 就直接作为长期系统使用。

## 安装前准备

需要准备：

- 一台小米游戏本八代 TIMI1801
- 一个 macOS 安装 U 盘
- OpenCore 基础知识
- ProperTree、OpenCore Configurator 或其他 plist 编辑工具
- GenSMBIOS 或等效工具，用来生成新的 SMBIOS

BIOS 建议：

- 关闭 Secure Boot
- 使用 AHCI 模式
- 关闭 CFG Lock，如 BIOS 无选项则保留当前 OpenCore 相关 Quirk
- 优先从 U 盘或 OpenCore EFI 启动

## 重要安全步骤

首次使用前必须重新生成 SMBIOS。打开 `EFI/OC/config.plist`，修改：

```text
PlatformInfo -> Generic -> SystemSerialNumber
PlatformInfo -> Generic -> MLB
PlatformInfo -> Generic -> SystemUUID
PlatformInfo -> Generic -> ROM
```

公开版占位符为：

```text
SystemSerialNumber = CHANGEME000000
MLB = CHANGEME0000000000000
SystemUUID = 00000000-0000-0000-0000-000000000000
ROM = <000000000000>
```

请生成自己的值后再登录 Apple ID。

## 安装方式

1. 制作 macOS 安装 U 盘。
2. 将本目录下的 `EFI/` 复制到安装 U 盘的 EFI 分区根目录。
3. 使用 GenSMBIOS 生成 `MacBookPro15,3` 对应的新序列号、MLB、UUID、ROM。
4. 写入 `EFI/OC/config.plist` 的 `PlatformInfo -> Generic`。
5. 用 `plutil -lint EFI/OC/config.plist` 或 ProperTree 检查 plist。
6. 从 U 盘 OpenCore 启动，安装 macOS。
7. 安装完成后，将同一份 EFI 复制到系统盘 EFI 分区。
8. 首次进入系统后安装 BetterDisplay，并按下方显示设置调整外接屏清晰度。

## 系统盘 EFI 写入流程

macOS 下不要假设磁盘编号固定。先确认当前系统物理盘：

```zsh
diskutil info /
diskutil info /dev/<APFS Physical Store>
diskutil info /dev/<whole disk>
```

确认目标硬盘是自己的 macOS 系统盘后，再挂载它的 EFI 分区：

```zsh
sudo diskutil mount /dev/<whole disk>s1
```

复制 EFI：

```zsh
sudo cp -R /Volumes/<USB_EFI>/EFI /Volumes/EFI/
```

建议复制前先备份原 EFI。

## 关键显示配置

Type-C 外接屏当前走 IGPU framebuffer con2：

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

稳定外接显示的关键项：

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

外接 AOC 24G1WG4 使用 `AAPL02,override-no-connect` 注入 256 字节 EDID。不要恢复旧的 `AAPL01` 注入。

## 外接屏文字清晰度

AOC 24G1WG4 是 1080p 显示器。macOS 原生 1080p 文本容易发虚。当前推荐：

- 保留并启动 BetterDisplay
- 外接屏使用 HiDPI 模式：实际渲染 `2880 x 1620`，界面看起来像 `1440 x 810 @ 60Hz`
- 字体平滑设置为 1

字体平滑命令：

```zsh
defaults -currentHost write -globalDomain AppleFontSmoothing -int 1
defaults write -globalDomain AppleFontSmoothing -int 1
defaults write -globalDomain CGFontRenderingFontSmoothingDisabled -bool NO
```

颜色配置建议使用系统为 `24V1W` 生成的 ColorSync 配置文件。不要强行切到 Display P3。

## 关键音频配置

Realtek ALC1220 当前使用 AppleALC layout 98，必须写成 4 字节 Data：

```text
PciRoot(0x0)/Pci(0x1f,0x3)
layout-id = <62000000>
alc-layout-id = <62000000>
```

`0x62` 的十进制是 `98`。

如需用命令写入，推荐：

```zsh
plutil -replace 'DeviceProperties.Add.PciRoot(0x0)/Pci(0x1f,0x3).layout-id' -data YgAAAA== EFI/OC/config.plist
plutil -replace 'DeviceProperties.Add.PciRoot(0x0)/Pci(0x1f,0x3).alc-layout-id' -data YgAAAA== EFI/OC/config.plist
```

不要使用 `PlistBuddy ... data 62000000`，它可能写成 ASCII 字节，导致音频设备消失。

## Wi-Fi 配置

EFI 中包含 `itlwm.kext`。公开版已经移除真实 Wi-Fi 信息，当前占位符为：

```text
EFI/OC/Kexts/itlwm.kext/Contents/Info.plist
WiFiConfig -> WiFi_1 -> ssid = CHANGE_ME_SSID
WiFiConfig -> WiFi_1 -> password = CHANGE_ME_PASSWORD
```

使用前请改成自己的网络信息，或改用 HeliPort 手动连接。

## 验证命令

进入系统后可以运行：

```zsh
system_profiler SPDisplaysDataType SPAudioDataType
ioreg -r -c IOHDACodecDevice -l | rg -n "IOHDACodecDevice|IOHDACodecVendorID|IOHDACodecAddress"
ioreg -p IOService -n HDEF -r -l | rg -n "layout-id|alc-layout-id|HDAConfigDefault|CodecName|PinConfigurations" -C 2
ioreg -p IOService -n IGPU -r -l | rg -n "framebuffer-con2|disable-agdc|force-online|complete-modeset|AAPL02,override-no-connect|audio-selector" -C 1
```

预期状态：

- 内屏正常亮
- AOC `24V1W` 外接屏正常亮
- BetterDisplay 下外接屏显示 `2880 x 1620`，界面看起来像 `1440 x 810`
- 内置或耳机音频有声音
- HDMI/DP 显示器音频可能仍不存在

## Codex 安装维护 Skill

本目录包含一个可复用的 Codex Skill：

```text
Skill/xiaomi-gaming-8th-hackintosh/
```

可复制到：

```text
~/.codex/skills/xiaomi-gaming-8th-hackintosh
```

之后可以在 Codex 中用：

```text
Use $xiaomi-gaming-8th-hackintosh to install or repair the Xiaomi Gaming Laptop 8th-gen Hackintosh EFI.
```

该 Skill 记录了完整安全流程、已知可用配置、失败路径和验证命令。

## 致谢

本 EFI 使用 OpenCore 及多个社区 kext。相关二进制文件版权与许可证归各自上游项目所有。
