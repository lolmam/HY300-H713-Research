# HY300 / H713 GSI Research Summary

This directory contains research and findings regarding the implementation of **Android 11 (SDK 30) GSIs** on the Allwinner H713 platform.

## Research Files

| File | Focus | Status |
| --- | --- | --- |
| **[GSI_DISCOVERIES.md](https://github.com/lolmam/HY300-H713-Research/blob/main/GSI/GSI_DISCOVERIES.md)** | **Core Research:** Hardware compatibility, HWC blockers, and bootloader status. | **Primary** |

---

## Key Findings

### 1. The "Mount-to-Blank" Phenomenon

The most significant hurdle identified during GSI testing.

* **Behavior**: The device successfully renders boot animation frames when the data partition is unmounted.
* **The Drop**: The moment `/data` is mounted and the Android framework attempts to initialize SystemUI/settings, the display drops to "backlight only" mode.
* **Inference**: A configuration conflict exists between the generic GSI framework and the proprietary Allwinner Hardware Composer (HWC), leading to a hardware timeout.

### 2. Partition Naming Mismatch (`UDISK`)

A fundamental conflict in how the GSI perceives the device's storage layout.

* **Reality**: The physical partition is named `UDISK` (`/dev/block/mmcblk0p26`), whereas the GSI expects `userdata`.
* **Impact**: System services and Zygote crash immediately because `/data` remains empty.
* **Workaround**: Manual mounting of `UDISK` to `/data` is required to stabilize the `system_server`.

### 3. Bootloader Status & Vulnerabilities

The device exhibits inconsistent enforcement of its "locked" state.

* **Bootloader Fastboot**: Despite reporting `unlocked: no`, it is effectively permissive and allows flashing physical partitions like `super` and `boot`.
* **Fastbootd (Userspace)**: Properly enforces the lock state and refuses modifications to logical partitions.
* **Unlock Support**: While `ro.oem_unlock_supported=1` is present, there is no functional UI toggle or standard unlock protocol implemented.
