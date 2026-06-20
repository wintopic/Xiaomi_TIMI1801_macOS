---
name: xiaomi-gaming-8th-hackintosh
description: Install, restore, maintain, and troubleshoot the Xiaomi Gaming Laptop 8th-gen Hackintosh/OpenCore EFI, including Type-C DP external display mapping for UGREEN CM136/AOC 24G1WG4, ALC1220 audio layout, BetterDisplay HiDPI clarity, backups, disk-safety checks, and post-install validation. Use when working on this machine's macOS/EFI configuration, copying/restoring EFI, diagnosing display/audio/USB/boot issues, or documenting current working Hackintosh settings.
---

# Xiaomi Gaming 8th Hackintosh

## Prime Directive

Treat this machine as a fragile, working Hackintosh. Preserve the latest stable boot path first, then make the smallest reversible change needed.

Never read or write the Samsung/Windows disk unless the user explicitly requests a read-only diagnostic. Never assume disk numbers are stable. Derive the current macOS physical disk from `diskutil info /`, verify the media name is the WD system disk, and state the exact target before mounting or editing any EFI.

## Source Of Truth

Read [references/current-build.md](references/current-build.md) before changing EFI, display mapping, audio, BetterDisplay, USB, or boot settings. It records the current working configuration, known-good backup paths, failed experiments, and validation commands.

The latest stable U-disk backup is:

`/Volumes/XIAOMI/备份/当前可用配置-20260621-063410`

Use it as the recovery source when the user asks to restore the current working EFI.

## Workflow

1. Identify the goal: install, restore, back up, validate, or troubleshoot.
2. Read the relevant section of `references/current-build.md`.
3. For any EFI write, first identify the current macOS APFS physical store and whole disk, verify the media name is `WDS500G1X0E-00AFY0`, and back up the existing EFI.
4. Write only the intended target: current WD system EFI or the U disk, as requested. Do not touch `/Volumes/Windows`, `/Volumes/X`, or any Samsung disk.
5. Validate with `plutil -lint`, targeted `ioreg`, `system_profiler`, and a reboot/test loop when required.
6. Keep a short note of what changed and where the backup was created.

## Editing Rules

- Use structured plist tools for `config.plist`; do not hand-edit binary-looking values casually.
- For audio Data values, prefer `plutil -replace ... -data <base64>`. Do not use `PlistBuddy ... data 62000000`; it can write ASCII bytes instead of 4-byte Data.
- Preserve the working Type-C DP mapping on IGPU framebuffer con2 unless the task is specifically to change display routing.
- Preserve ALC1220 audio layout `98` as 4-byte Data in config unless troubleshooting audio.
- Preserve BetterDisplay for external monitor text clarity.
- Avoid DDC/DDC-CI attempts over the Type-C-to-HDMI adapter; it previously froze the system.

## Validation Quick Commands

Use targeted commands instead of huge unrestricted dumps:

```zsh
system_profiler SPDisplaysDataType SPAudioDataType
ioreg -r -c IOHDACodecDevice -l | rg -n "IOHDACodecDevice|IOHDACodecVendorID|IOHDACodecAddress"
ioreg -p IOService -n HDEF -r -l | rg -n "layout-id|alc-layout-id|CodecList|HDAConfigDefault|CodecName|PinConfigurations" -C 2
ioreg -p IOService -n IGPU -r -l | rg -n "framebuffer-con2|disable-agdc|force-online|complete-modeset|AAPL02,override-no-connect|hda-gfx" -C 1
```

If a change affects boot, display, or audio, ask the user to reboot and then run validation again.
