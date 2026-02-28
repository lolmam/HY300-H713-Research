# HY300 Pro+ (Magcubic) – Full Firmware Dump & Root Guide

## Device Overview

* Model: HY300 Pro+
* SoC: Allwinner H713 (sun50iw12p1)
* Android Version: 11
* Architecture:

  * Kernel: ARM64
  * Userspace: armeabi-v7a (32-bit)
* Partition Layout: A/B (Seamless Updates)
* Storage: ~7.3GB eMMC
* Bootloader: U-Boot (Allwinner)


# Phase 1 — Hardware & Low-Level Exploration

## 1. Opened Device & Inspected PCB

Identified:

* Allwinner H713 SoC
* Debug pads (UART, FEL)
* U-Boot recovery button
* HDMI, WiFi, speaker outputs

## 2. Confirmed FEL Mode

Connected via USB and verified:

```bash
lsusb
```

Detected:

```
Allwinner Technology sunxi SoC OTG connector in FEL mode
```

FEL communication tested using:

```
sunxi-fel version
```

Confirmed working communication.

# Phase 2 — Full eMMC Firmware Dump (Via ADB)

## 1. Identified Full Disk

```bash
adb shell cat /proc/partitions
adb shell cat /sys/block/mmcblk0/size
```

Total sectors:

```
15269888
```

Total size:

```
7,818,182,656 bytes (~7.3GB)
```

## 2. Full Raw Dump

Dumped full eMMC user area:

```bash
adb shell
dd if=/dev/block/mmcblk0 of=/storage/USB_ID/emmc_full.img bs=1M
```

Result:

```
7818182656 bytes copied
```

Confirmed full raw image.

## 3. Verified GPT Partition Table

On Ubuntu:

```bash
gdisk -l emmc_full.img
```

Valid GPT detected with partitions:

* bootloader_a
* bootloader_b
* boot_a
* boot_b
* vendor_boot_a/b
* super
* vbmeta_a/b
* userdata (UDISK)
* etc.

This confirmed:

* Proper A/B layout
* No corruption
* Complete backup

# Phase 3 — Boot Image Extraction

From GPT table:

boot_a:

* Start sector: 205824
* End sector: 336895
* Size: 64MB

Extracted boot image:

```bash
dd if=emmc_full.img of=boot_a.img bs=512 skip=205824 count=131072
```

Verified:

```
Android bootimg
```

---

# Phase 4 — Rooting via Magisk (Safe A/B Method)

## 1. Confirm Active Slot

```bash
adb shell getprop ro.boot.slot_suffix
```

Output:

```
_a
```

Active slot = boot_a

## 2. Patched Boot Image

Steps:

1. Installed Magisk APK
2. Pushed boot_a.img to device
3. Patched inside Magisk
4. Pulled patched image:

```
boot_a_patched.img
```

## 3. Fastboot Flash

Rebooted to bootloader:

```bash
adb reboot bootloader
```

Verified fastboot:

```bash
fastboot devices
```

Detected device successfully.

Checked unlock state:

```bash
fastboot getvar unlocked
```

Returned:

```
unlocked: not supported
```

Indicates:

* No strict bootloader lock enforcement.

Flashed patched boot:

```bash
fastboot flash boot_a boot_a_patched.img
```

Result:

```
OKAY
Finished.
```

Rebooted:

```bash
fastboot reboot
```

# Phase 5 — Root Confirmed

```bash
adb shell
su
whoami
```

Output:

```
root
```

Magisk shows installed and active.

Root successfully achieved.

# Current Device State

* Full raw firmware backup
* Verified GPT layout
* Fastboot functional
* A/B slots intact
* Working root (Magisk)
* FEL access confirmed
* Safe rollback image stored

# Safety Net

If anything breaks:

* Reflash original boot image or any other stock dump image via fastboot
* Or restore full eMMC via ADB if su available:

```bash
dd if=emmc_full.img of=/dev/block/mmcblk0 bs=4M
```

# Result

Device moved from:

Restricted vendor Android build

To:

Fully owned, root-accessible, backed-up hardware platform.

Total control achieved.
