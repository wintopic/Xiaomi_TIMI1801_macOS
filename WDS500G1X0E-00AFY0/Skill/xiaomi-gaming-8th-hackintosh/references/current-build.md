# Current Build Reference

## Table Of Contents

- Safety and backup map
- Machine profile
- Current stable EFI
- ACPI, drivers, and kexts
- IGPU and Type-C DP display mapping
- External display clarity
- Audio
- Networking, Bluetooth, USB, input, and storage
- Known bad paths
- Safe restore and backup procedure
- Validation checklist

## Safety And Backup Map

- Do not touch the Samsung/Windows disk. In recent sessions it appeared as `/Volumes/Windows` and as an NTFS disk; disk numbers change across boots.
- The current macOS system disk is the WD NVMe media `WDS500G1X0E-00AFY0`. Its EFI is the first partition of the current whole disk, but the identifier may be `disk0s1` or `disk1s1` depending on boot order.
- The U disk is `XIAOMI`, usually mounted at `/Volumes/XIAOMI`.
- The latest stable backup is `/Volumes/XIAOMI/备份/当前可用配置-20260621-063410`.
- That backup contains `EFI/`, a standalone `config.plist`, `说明.txt`, and detection snapshots under `检测/`.
- Older important backup: `/Volumes/XIAOMI/EFI_BACKUPS/current-system-disk1s1-外接显示器可用版-20260620-032212`.

Before writing EFI, derive the target:

```zsh
diskutil info /
diskutil info /dev/<APFS Physical Store>
diskutil info /dev/<whole disk>
```

Only proceed if the whole disk media name is `WDS500G1X0E-00AFY0`.

## Machine Profile

- Laptop: Xiaomi Gaming Laptop 8th generation.
- SMBIOS: `MacBookPro15,3`.
- macOS at capture: `15.7.7 (24G720)`.
- CPU exposed by macOS: 6-core Intel Core i7 at 2.2 GHz, matching the i7-8750H class.
- Memory: 16 GB.
- iGPU: Intel UHD Graphics 630, device id `0x3e9b`.
- dGPU: disabled with `SSDT-Disable_GPU_PEG0.aml`.
- Internal display: `LM156LF9L01`, 1920 x 1080 at 60 Hz.
- External display tested stable: AOC 24G1WG4, EDID name `24V1W`, hardware id from Windows log `Monitor\AOC2401`.
- Adapter: UGREEN CM136-70495 USB-C multifunction adapter.
- System disk: WD `WDS500G1X0E-00AFY0`.
- Audio controller: Intel Cannon Lake-H/S cAVS `8086:a348`.
- Audio codec: Realtek ALC1220 `10ec:1220`.
- Display audio codec seen intermittently: Intel Display Audio `8086:280b`.

Do not copy real serial numbers, MLB, ROM, UUID, or provisioning IDs from this machine into public docs or another installation. Generate fresh SMBIOS identifiers for a reinstall.

## Current Stable EFI

Stable source:

`/Volumes/XIAOMI/备份/当前可用配置-20260621-063410/EFI`

OpenCore drivers:

- `HfsPlus.efi`
- `OpenRuntime.efi`
- `ResetNvramEntry.efi`

SMBIOS:

- `PlatformInfo -> Automatic = true`
- `SystemProductName = MacBookPro15,3`
- Keep serial-like identifiers private and regenerate if reinstalling.

Important NVRAM boot args:

```text
-v debug=0x100 keepsyms=1 -igfxblt igfxonln=1 -igfxtypec -igfxlspcon igfxonlnfbs=2 igfxfcms=1 igfxfcmsfbs=2 igfxagdc=0
```

SIP config in backup:

```text
csr-active-config = <030a0000>
```

## ACPI, Drivers, And Kexts

Enabled ACPI:

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

Enabled core kexts and captured versions:

- `Lilu.kext` 1.7.3
- `WhateverGreen.kext` 1.7.1
- `AppleALC.kext` 1.9.8
- `VirtualSMC.kext` 1.3.8
- `SMCBatteryManager.kext` 1.3.8
- `SMCLightSensor.kext` 1.3.8
- `SMCProcessor.kext` 1.3.8
- `SMCSuperIO.kext` 1.3.8
- `NVMeFix.kext` 1.1.4
- `RestrictEvents.kext` 1.1.7
- `USBToolBox.kext` 1.2.0
- `UTBMap.kext` 1.1
- `XHCI-unsupported.kext` 0.9.2
- `IntelBluetoothFirmware.kext` 2.5.0
- `IntelBTPatcher.kext` 2.5.0
- `BlueToolFixup.kext` 2.7.3, `MinKernel = 21.0.0`
- `itlwm.kext` 2.3.0
- `RealtekRTL8111.kext` 2.4.2
- `BrightnessKeys.kext` 1.0.4
- `Sinetek-rtsx.kext` 9.0.0
- `VoodooPS2Controller.kext` 2.3.8

Disabled in config:

- `VoodooI2C.kext`
- `VoodooI2CHID.kext`
- `VoodooI2CServices.kext`
- `VoodooGPIO.kext`
- `VoodooInput.kext` under VoodooI2C
- `VoodooInput.kext` under VoodooPS2Controller
- `VoodooPS2Trackpad.kext`

Enabled under VoodooPS2:

- `VoodooPS2Controller.kext`
- `VoodooPS2Keyboard.kext`
- `VoodooPS2Mouse.kext`

## IGPU And Type-C DP Display Mapping

Working route:

- Type-C display output maps to `AppleIntelFramebuffer@2`.
- Windows ACPI path observed: `ACPI(_SB_)#ACPI(PCI0)#ACPI(GFX0)#ACPI(DD02)`.
- Use IGPU device path `PciRoot(0x0)/Pci(0x2,0x0)`.

Core IGPU properties:

```text
AAPL,ig-platform-id = <0900a53e>
device-id runtime = <9b3e0000>
framebuffer-patch-enable = <01000000>
framebuffer-portcount = <03000000>
framebuffer-pipecount = <03000000>
framebuffer-stolenmem = <00003001>
framebuffer-fbmem = <00009000>
hda-gfx = "onboard-1"
```

Internal panel con0:

```text
framebuffer-con0-enable = <01000000>
framebuffer-con0-index = <00000000>
framebuffer-con0-busid = <00000000>
framebuffer-con0-pipe = <08000000>
framebuffer-con0-type = <02000000>
framebuffer-con0-flags = <98000000>
```

Other DP con1:

```text
framebuffer-con1-enable = <01000000>
framebuffer-con1-index = <01000000>
framebuffer-con1-busid = <05000000>
framebuffer-con1-pipe = <09000000>
framebuffer-con1-type = <00040000>
framebuffer-con1-flags = <87010000>
```

Working Type-C/AOC con2:

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

Online/modeset stability:

```text
disable-agdc = <01000000>
force-online = <01000000>
force-online-framebuffers = <0200000000000000>
complete-modeset = <01000000>
complete-modeset-framebuffers = <0200000000000000>
```

EDID:

- Inject only `AAPL02,override-no-connect`.
- Do not restore older `AAPL01` injection.
- Current injected EDID is 256 bytes and includes a CTA extension. It enabled macOS to read the AOC/24V1W display more consistently.
- Runtime display name: `24V1W`.
- Runtime connection type: DVI or HDMI via the UGREEN adapter.

## External Display Clarity

Current clear-text setup:

- BetterDisplay is installed at `/Applications/BetterDisplay.app` and should remain installed/running.
- External display runtime mode after BetterDisplay: physical render `2880 x 1620`, UI looks like `1440 x 810 @ 60Hz`.
- This HiDPI-like mode is important for clearer text on the 1080p AOC panel. Without BetterDisplay, macOS will likely fall back to ordinary 1920 x 1080 and text will look less sharp.

Font settings:

```zsh
defaults -currentHost write -globalDomain AppleFontSmoothing -int 1
defaults write -globalDomain AppleFontSmoothing -int 1
defaults write -globalDomain CGFontRenderingFontSmoothingDisabled -bool NO
```

Known display guidance:

- Keep the system-generated ColorSync profile for `24V1W` unless colors are visibly wrong.
- Do not switch to Display P3 for this monitor.
- Use sRGB only if colors look oversaturated.
- Avoid DDC/DDC-CI control over the Type-C-to-HDMI adapter; DDC once read brightness but later froze the system.

## Audio

Working internal audio uses AppleALC ALC1220 layout 98, encoded as 4-byte Data in `config.plist`.

Config path:

`DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x1f,0x3)`

Stable config values:

```text
layout-id = <62000000>
alc-layout-id = <62000000>
```

`0x62` decimal is `98`.

Use `plutil -replace ... -data YgAAAA==` to write this value. Do not use `PlistBuddy ... data 62000000`; that can write ASCII bytes and break audio matching.

Runtime notes:

- HDEF may still show `layout-id = <07000000>` in IOReg while `alc-layout-id = <62000000>`.
- AppleALC's loaded HDAConfigDefault should identify `Custom ALC1220 for Mi Gaming Notebook Creator by Xsixu`.
- Audio output currently works through the internal/headphone path. In one capture `system_profiler` reported `Built-in Output` with output source `Headphones`.

Known not-yet-working:

- AOC/HDMI/DP monitor audio is not currently exposed as a CoreAudio output.
- The Intel Display Audio codec `8086:280b` appears intermittently but has not reliably bound to AppleGFXHDA.

Failed audio attempts:

- Layout 7 enumerated devices but produced no sound and previously triggered an AppleHDA/DspFuncLib panic.
- Writing layout 98 as an integer caused `alc-layout-id` and `layout-id` mismatch and produced an empty audio device list.
- The correct fix was writing both properties as Data `<62000000>` in config.

## Networking, Bluetooth, USB, Input, And Storage

Network-related kexts in current EFI:

- `itlwm.kext` for Intel Wi-Fi.
- `IntelBluetoothFirmware.kext`, `IntelBTPatcher.kext`, `BlueToolFixup.kext` for Bluetooth.
- `RealtekRTL8111.kext` for Realtek Ethernet.

USB stack:

- `USBToolBox.kext`
- `UTBMap.kext`
- `XHCI-unsupported.kext`
- `SSDT-USBX.aml`

Storage:

- `NVMeFix.kext` enabled.
- Current macOS disk is WD `WDS500G1X0E-00AFY0`.

Input:

- PS/2 keyboard and mouse plugins are enabled.
- I2C stack is present but disabled in config.

## Known Bad Paths

- Do not touch Samsung/Windows disk.
- Do not run DDC/DDC-CI tests unless the user explicitly accepts freeze risk.
- Do not replace the working con2 mapping with generic connector guesses.
- Do not remove BetterDisplay if the user cares about external monitor text clarity.
- Do not use ALC1220 layout 7 as the final audio layout; it was not audible.
- Do not write plist Data with PlistBuddy's `data` argument unless verified; use `plutil -data` or a structured plist library.
- Do not inject `AAPL01,override-no-connect`; current stable display uses only `AAPL02`.

## Safe Restore And Backup Procedure

For restore from U disk backup to current WD system EFI:

1. Verify `/Volumes/XIAOMI/备份/当前可用配置-20260621-063410/EFI` exists.
2. Derive the current macOS whole disk from `diskutil info /`.
3. Verify the whole disk media name is `WDS500G1X0E-00AFY0`.
4. Mount `<whole disk>s1`.
5. Back up the current target EFI to a timestamped folder before replacing anything.
6. Copy the backup `EFI/` to the mounted EFI volume.
7. Run `plutil -lint` on `EFI/OC/config.plist`.
8. Unmount the EFI and reboot.

Use `ditto` for EFI directory copies on macOS.

For backup to U disk:

1. Create a timestamped folder under `/Volumes/XIAOMI/备份/`.
2. Copy the full mounted `EFI/` directory.
3. Copy standalone `config.plist`.
4. Save validation snapshots: display, audio, NVRAM, HDEF, IGPU, and CoreAudio device list.
5. Unmount the system EFI.

## Validation Checklist

After a restore or EFI edit:

```zsh
plutil -lint /Volumes/EFI/EFI/OC/config.plist
system_profiler SPDisplaysDataType SPAudioDataType
nvram -p | rg "boot-args|csr-active-config"
ioreg -r -c IOHDACodecDevice -l | rg -n "IOHDACodecDevice|IOHDACodecVendorID|IOHDACodecAddress"
ioreg -p IOService -n HDEF -r -l | rg -n "layout-id|alc-layout-id|HDAConfigDefault|CodecName|PinConfigurations" -C 2
ioreg -p IOService -n IGPU -r -l | rg -n "framebuffer-con2|disable-agdc|force-online|complete-modeset|AAPL02,override-no-connect|audio-selector" -C 1
```

Expected user-visible state:

- Internal display works.
- AOC `24V1W` external display lights through the UGREEN CM136 adapter.
- BetterDisplay provides the external HiDPI mode `2880 x 1620`, UI looks like `1440 x 810`.
- Internal/headphone audio plays sound with ALC1220 layout 98.
- HDMI/DP monitor audio may still be absent; do not treat that as a regression unless it was explicitly fixed later.
