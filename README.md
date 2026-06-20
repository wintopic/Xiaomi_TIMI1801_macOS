# Xiaomi_TIMI1801_macOS

小米游戏本八代 TIMI1801 的 macOS / OpenCore 配置仓库。

本项目整理了一份已经实机验证的 EFI、安装说明、硬件功能状态、诊断快照和 Codex 维护 Skill，适用于小米游戏本八代平台的黑苹果安装、恢复和后续调试。

## 项目内容

```text
.
├── README.md
└── WDS500G1X0E-00AFY0/
    ├── EFI/                         # OpenCore EFI
    ├── config.plist                  # 独立配置副本
    ├── Diagnostics/                  # 实机检测快照
    ├── Skill/                        # Codex 安装维护 Skill
    └── README.md                     # 详细安装与硬件说明
```

当前配置按来源系统盘型号分目录保存：

- [WDS500G1X0E-00AFY0](./WDS500G1X0E-00AFY0)

## 已验证硬件

| 项目 | 信息 |
| --- | --- |
| 机型 | 小米游戏本八代 TIMI1801 |
| SMBIOS | `MacBookPro15,3` |
| CPU | Intel Core i7 六核，i7-8750H 同级 |
| 核显 | Intel UHD Graphics 630 |
| 声卡 | Intel HDA `8086:a348` + Realtek ALC1220 |
| 内屏 | `LM156LF9L01`，1920 x 1080 @ 60Hz |
| 已验证外接屏 | AOC 24G1WG4，EDID 名称 `24V1W` |
| 已验证转接器 | UGREEN CM136-70495 USB-C 多功能转换器 |
| 来源系统盘 | WD `WDS500G1X0E-00AFY0` |

## 功能状态概览

| 功能 | 状态 |
| --- | --- |
| macOS 启动 | 可用 |
| 核显加速 | 可用 |
| 内屏显示 | 可用 |
| Type-C 外接显示 | 可用 |
| BetterDisplay HiDPI 外接屏清晰文字 | 可用 |
| 内置/耳机音频 | 可用 |
| Wi-Fi | 配置已包含 |
| 蓝牙 | 配置已包含 |
| 有线网卡 | 配置已包含 |
| USB 映射 | 配置已包含 |
| 独显 | 已禁用 |
| HDMI/DP 显示器音频 | 未稳定启用 |

## 快速开始

1. 进入 [WDS500G1X0E-00AFY0](./WDS500G1X0E-00AFY0)。
2. 阅读目录内的完整安装说明。
3. 复制 `EFI/` 到安装 U 盘或系统盘 EFI 分区。
4. 使用 GenSMBIOS 生成自己的 `MacBookPro15,3` SMBIOS 信息，并写入 `EFI/OC/config.plist`。
5. 如使用 `itlwm.kext`，按 README 修改 Wi-Fi 占位符。
6. 安装 macOS 后，按说明安装 BetterDisplay 并设置外接屏 HiDPI。

## 文档入口

完整安装流程、BIOS 建议、关键显示配置、音频配置、验证命令和已知问题见：

[WDS500G1X0E-00AFY0/README.md](./WDS500G1X0E-00AFY0/README.md)
