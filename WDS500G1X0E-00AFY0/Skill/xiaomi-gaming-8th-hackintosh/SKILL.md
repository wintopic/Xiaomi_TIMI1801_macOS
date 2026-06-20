---
name: xiaomi-gaming-8th-hackintosh
description: Install, restore, maintain, and troubleshoot the Xiaomi Gaming Laptop 8th-gen Hackintosh/OpenCore EFI, including Type-C DP external display mapping for UGREEN CM136/AOC 24G1WG4, ALC1220 audio layout, BetterDisplay HiDPI clarity, backups, hardware notes, and post-install validation. Use when working on this machine's macOS/EFI configuration, copying/restoring EFI, diagnosing display/audio/USB/boot issues, or documenting current working Hackintosh settings.
---

# Xiaomi Gaming 8th Hackintosh

## Maintenance Approach

Preserve the latest stable boot path first, then make the smallest reversible change needed. Disk identifiers can change between boots, so derive the current macOS physical disk from `diskutil info /`, verify the expected media name, and state the exact target before mounting or editing any EFI.

## Source Of Truth

Read [references/current-build.md](references/current-build.md) before changing EFI, display mapping, audio, BetterDisplay, USB, or boot settings. It records the current working configuration, known-good backup paths, failed experiments, and validation commands.

The latest stable U-disk backup is:

`/Volumes/XIAOMI/备份/当前可用配置-20260621-063410`

Use it as the recovery source when the user asks to restore the current working EFI.

## Pitfall Checklist

Use this checklist before any install, restore, or troubleshooting change:

1. Identify the actual target disk with `diskutil info /`, then inspect the APFS physical store and whole disk. Do not trust `disk0`, `disk1`, or boot-order guesses.
2. If the task is to update the captured working build, the expected system disk media name is `WDS500G1X0E-00AFY0`. If the media name differs, stop and ask for confirmation before writing EFI.
3. Back up the target EFI before replacing it. Keep the latest stable U-disk EFI untouched until the new system EFI has booted successfully.
4. Replace public placeholders before first boot: generate a fresh `MacBookPro15,3` SMBIOS and fill Wi-Fi information only on the user's local copy.
5. Do not commit real serial numbers, MLB values, UUIDs, ROM values, Wi-Fi SSIDs, Wi-Fi passwords, or local-only backup dumps.
6. Preserve the Type-C display route on IGPU `framebuffer-con2` for UGREEN CM136 and AOC 24G1WG4. Keep the AOC EDID override on `AAPL02,override-no-connect`; do not restore older `AAPL01` overrides.
7. Write ALC1220 layout 98 as 4-byte Data (`YgAAAA==`, shown as `<62000000>`). Do not write it as an integer or as ASCII text.
8. Keep BetterDisplay installed and configured for external-monitor text clarity. Without it, the 1080p AOC panel will usually fall back to less sharp native scaling.
9. Avoid DDC/DDC-CI experiments through the Type-C-to-HDMI adapter; this route previously froze the system.
10. Treat missing HDMI/DP monitor audio as a known limitation unless a later validated build says otherwise. Do not destabilize working internal/headphone audio while chasing display-audio output.

## Workflow

1. Identify the goal: install, restore, back up, validate, or troubleshoot.
2. Read the relevant section of `references/current-build.md`.
3. For any EFI write, first identify the current macOS APFS physical store and whole disk, verify the media name is `WDS500G1X0E-00AFY0`, and back up the existing EFI.
4. Write only the intended EFI target: the current macOS system EFI or the prepared U-disk EFI.
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
