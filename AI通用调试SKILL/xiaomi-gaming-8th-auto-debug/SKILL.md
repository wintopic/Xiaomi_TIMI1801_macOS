---
name: xiaomi-gaming-8th-auto-debug
description: AI-assisted Hackintosh/OpenCore debugging and EFI migration workflow for Xiaomi Gaming Laptop 8th generation TIMI1801 variants, especially when the target user has a different SSD, monitor, Type-C adapter, Wi-Fi/Bluetooth card, audio state, BIOS state, or local EFI. Use when adapting the Xiaomi TIMI1801 reference EFI, diagnosing boot/display/audio/USB/network issues, safely identifying the correct EFI target, backing up before edits, validating config.plist, and avoiding machine-specific assumptions.
---

# Xiaomi Gaming 8th Auto Debug

## Scope

Use this skill for Xiaomi Gaming Laptop 8th generation TIMI1801 Hackintosh work. It is a general debugging workflow for this laptop family, not a universal EFI recipe for unrelated computers.

The repository's `WDS500G1X0E-00AFY0` EFI is a known-good reference build. Treat its values as evidence from one working TIMI1801 machine. Do not assume every user has the same SSD model, external monitor, Type-C adapter, Wi-Fi credentials, USB behavior, BIOS state, or EDID.

## Core Rule

Preserve a known-good boot path first. Then identify the target hardware and make the smallest reversible change needed to test one hypothesis.

Before any EFI write, state the exact target disk, EFI volume, backup path, planned file change, and expected validation signal. If the disk or EFI target is unclear, stop and ask for confirmation.

## TIMI1801 Reference Baseline

Use these as starting points to compare against the user's actual machine:

- Laptop family: Xiaomi Gaming Laptop 8th generation TIMI1801.
- Typical CPU/iGPU: Coffee Lake-H class CPU with Intel UHD Graphics 630.
- Typical SMBIOS: `MacBookPro15,3`.
- Typical audio path: Intel HDA controller with Realtek ALC1220, AppleALC layout 98.
- Typical dGPU handling: NVIDIA dGPU disabled through ACPI.
- Known working Type-C case in this repo: UGREEN CM136-70495 to AOC 24G1WG4, EDID name `24V1W`, IGPU framebuffer con2.
- Known clarity fix for 1080p external monitor: BetterDisplay HiDPI.

Always re-check these facts on the target machine. A different SSD name is normal and must not be treated as an error.

## Before Any Write

1. Identify all disks:

```zsh
diskutil list
diskutil info /
```

2. From `diskutil info /`, find the APFS physical store, then inspect the physical store and whole disk:

```zsh
diskutil info /dev/<APFS Physical Store>
diskutil info /dev/<whole disk>
```

3. State the target media name and EFI partition. Do not trust `disk0`, `disk1`, or old notes.
4. Mount only the intended EFI partition.
5. Back up the mounted target EFI to a timestamped folder before editing or replacing it.
6. Keep at least one bootable USB or known-good EFI unchanged until the edited EFI survives a reboot.
7. Validate every plist edit:

```zsh
plutil -lint EFI/OC/config.plist
```

## Hardware Discovery

Collect current facts before selecting patches or kexts. Prefer targeted commands and user screenshots/logs over assumptions.

macOS commands:

```zsh
system_profiler SPHardwareDataType SPDisplaysDataType SPAudioDataType SPUSBDataType SPNVMeDataType SPPCIDataType SPNetworkDataType SPBluetoothDataType
ioreg -p IOService -n IGPU -r -l
ioreg -p IOService -n HDEF -r -l
ioreg -r -c IOHDACodecDevice -l
nvram -p | rg "boot-args|csr-active-config|prev-lang:kbd"
```

Windows commands and tools are useful when a device works in Windows but not macOS:

```powershell
Get-PnpDevice -Class Monitor | Format-List *
Get-PnpDevice -Class Display | Format-List *
Get-PnpDevice -Class Media | Format-List *
Get-PnpDevice -Class Net | Format-List *
Get-PnpDevice -Class Bluetooth | Format-List *
```

Also use HWiNFO64, Device Manager hardware IDs, EDID dumps, and OpenCore boot logs when available.

## Migration Workflow

1. Confirm the machine is a Xiaomi Gaming Laptop 8th generation TIMI1801 or a very close board variant.
2. Create a working folder named after the target machine, SSD model, or user's hardware profile.
3. Copy the reference EFI into that working folder, not directly onto the system EFI.
4. Replace public placeholders with local values:
   - Fresh `MacBookPro15,3` SMBIOS.
   - Local ROM value.
   - Local Wi-Fi details only if using embedded `itlwm` configuration.
5. Compare the target hardware map with the reference EFI:
   - `ACPI`
   - `DeviceProperties`
   - `Kernel -> Add`
   - `Kernel -> Quirks`
   - `NVRAM -> Add -> boot-args`
   - `PlatformInfo`
   - `UEFI -> Drivers`
6. Change one variable at a time. Record old value, new value, plist path, and reason.
7. Validate with targeted commands before asking for a reboot.
8. After reboot, compare runtime evidence with the expected result and keep only improvements.

## Boot And Storage

- Different SSD models are expected. Derive the target disk from the running system instead of searching for a fixed WD model.
- Keep `NVMeFix.kext` when the machine uses NVMe storage unless evidence says it causes trouble.
- For a new SSD install, write the EFI to the new SSD only after the USB EFI boots the installer or installed system.
- If OpenCore appears but macOS does not boot, inspect OpenCore version, `OpenRuntime.efi`, APFS visibility, SecureBootModel, kernel quirks, and boot args.
- If the system cannot boot after a change, restore the last known-good EFI from the timestamped backup or USB.

## Display And Type-C

- Start with the TIMI1801 iGPU path `PciRoot(0x0)/Pci(0x2,0x0)`, then verify runtime IOReg.
- Internal display should load through Intel UHD Graphics 630 with acceleration.
- The repo's known working Type-C path uses IGPU framebuffer con2 for UGREEN CM136 and AOC 24G1WG4. Use it as a strong hint, not as proof for every adapter.
- For another monitor or adapter, collect EDID, Windows monitor hardware ID, connection path, and macOS IOReg before changing connector values.
- Treat busid, pipe, connector type, connector flags, LSPCON, force-online, complete-modeset, and EDID injection as machine/adapter-specific hypotheses.
- Use only the EDID override that matches the actual connector. Do not copy unrelated `AAPL01` or `AAPL02` overrides blindly.
- If the external display lights but text is blurry, solve clarity separately with HiDPI/BetterDisplay or suitable scaling.

## Audio

- Verify the audio controller and codec with `ioreg`; do not assume the layout until the codec is confirmed.
- The reference TIMI1801 build uses Realtek ALC1220 with AppleALC layout 98.
- Write layout 98 as 4-byte Data, shown as `<62000000>` and base64 `YgAAAA==`:

```zsh
plutil -replace 'DeviceProperties.Add.PciRoot(0x0)/Pci(0x1f,0x3).layout-id' -data YgAAAA== EFI/OC/config.plist
plutil -replace 'DeviceProperties.Add.PciRoot(0x0)/Pci(0x1f,0x3).alc-layout-id' -data YgAAAA== EFI/OC/config.plist
```

- Do not write audio layout Data as ASCII text or casual plist text.
- Do not switch to another layout just because devices enumerate; require actual audible output.
- Treat HDMI/DP monitor audio as a separate feature from internal/headphone audio.

## USB, Network, And Bluetooth

- USB maps can differ with BIOS, board revision, internal Bluetooth module, and Type-C behavior. Verify before treating a map as universal.
- If Bluetooth is unstable, confirm the internal USB port is mapped correctly.
- Identify Wi-Fi/Bluetooth chipset before selecting `itlwm`, AirportItlwm, Broadcom patches, or Intel Bluetooth firmware kexts.
- Do not publish real SSIDs or passwords. Public `itlwm` configs should use placeholders.
- If Ethernet differs, verify the actual PCI ID before changing Realtek or Intel Ethernet kexts.

## Common TIMI1801 Pitfalls

- Writing to the wrong EFI because disk numbers changed.
- Installing directly to a new SSD before proving the USB EFI can boot.
- Reusing someone else's SMBIOS, ROM, UUID, serial number, Wi-Fi SSID, or password.
- Treating the repo's WD disk name as required hardware.
- Treating the repo's AOC EDID as universal for all monitors.
- Breaking working internal display while chasing Type-C output.
- Breaking working internal/headphone audio while chasing monitor audio.
- Writing `layout-id` as the wrong plist type.
- Replacing model-specific SSDTs or USB maps without comparing against the target TIMI1801 hardware.
- Running risky DDC/DDC-CI tests through adapters without a recovery path.

## Validation Commands

Display:

```zsh
system_profiler SPDisplaysDataType
ioreg -p IOService -n IGPU -r -l | rg -n "AAPL|framebuffer|connector|con[0-9]|EDID|display|online|modeset|agdc|lspcon" -C 1
```

Audio:

```zsh
system_profiler SPAudioDataType
ioreg -r -c IOHDACodecDevice -l | rg -n "IOHDACodecDevice|IOHDACodecVendorID|IOHDACodecAddress"
ioreg -p IOService -n HDEF -r -l | rg -n "layout-id|alc-layout-id|CodecList|HDAConfigDefault|CodecName|PinConfigurations" -C 2
```

USB and network:

```zsh
system_profiler SPUSBDataType SPNetworkDataType SPBluetoothDataType
ioreg -p IOUSB -l | rg -n "AppleUSB|XHC|HS[0-9]|SS[0-9]|Bluetooth" -C 1
```

## Response Style For AI Operators

Before changing anything, report:

- Confirmed target machine and hardware differences from the reference build.
- Target EFI and disk identity.
- Backup path.
- Exact planned change.
- Expected validation signal.

After changing anything, report:

- Files changed.
- Values changed.
- Validation commands run and results.
- Whether a reboot is required.
- Rollback path.
