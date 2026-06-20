# Xiaomi_TIMI1801_macOS

小米游戏本八代 TIMI1801 黑苹果 OpenCore 配置归档。

当前已发布的稳定配置按硬盘型号分目录存放：

- [WDS500G1X0E-00AFY0](./WDS500G1X0E-00AFY0)

进入对应目录查看完整安装说明、EFI、诊断快照和 Codex 安装维护 Skill。

## 安全说明

公开仓库里的 `config.plist` 已经移除真实 SMBIOS 信息。首次使用前必须重新生成并填写：

- `SystemSerialNumber`
- `MLB`
- `SystemUUID`
- `ROM`

不要直接使用他人的序列号登录 Apple ID。

