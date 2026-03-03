# GSI on HY300 Pro H713 Research

## 1. Core Hardware & Environment
- **SoC:** Allwinner H713 (Platform: `ares`, Board: `exdroid`, Hardware: `sun50iw12p1`). (Reported from GSI ADB Shell)
- **Arch:** 32-bit ARM (`armeabi-v7a`).
- **GSI Used:** Android 11 (SDK 30) with PHH Treble patches.
- **Projector Specifics:** Uses a secondary "MIPS" processor for display tasks (keystone, focus) and a proprietary `sunxihwc` Hardware Composer.

## 2. Major Discoveries

### A. Partition Naming Mismatch
- **Observation:** The GSI expected `/dev/block/by-name/userdata`.
- **Reality:** The physical partition is named `UDISK` (`/dev/block/mmcblk0p26`).
- **Impact:** `init` could not perform `post-fs-data`, meaning `/data` remained empty. This caused Zygote and all system services to crash immediately because they couldn't create Dalvik caches or access settings.
- **Workaround:** Manual mounting of `UDISK` to `/data` allowed the framework (`system_server`) to stabilize, and manually triggered `post-fs-data` using `setprop vold.decrypt trigger_post_fs_data`.

### B. The "Mount-to-Blank" Phenomenon
- **Discovery:** This is the most significant behavior observed.
  - **Data Unmounted:** The device shows an "empty image" on the wall (a rendered black/grey frame from the boot animation). This proves the Display Engine and HWC *can* work.
  - **Data Mounted:** The moment the data partition is mounted and the Android framework attempts to load user settings/SystemUI, the display immediately drops to "backlight only" mode.
- **Inference:** There is a conflict between the vendor's proprietary display settings stored in `/data` (or expected by the HAL) and the GSI's framework. It is likely that `SystemUI` or `ActivityManager` sends a configuration that the Allwinner HWC cannot handle, leading to a hardware timeout.

### C. Hardware Composer (HWC) Blockers
- **Config Files:** The vendor HWC requires specific JSON files in `/vendor/etc/dispconfigs/`. It specifically looks for `h713-tuna_p3.json` or `config.json`.
- **Client Failure:** `SurfaceFlinger` frequently logs `failed to create composer client`. This indicates a version mismatch or a security/permission failure between the GSI's generic SurfaceFlinger and the Allwinner-specific HWC HAL.
- **Timeouts:** Even when connected, the logs show `waitForCommitFinish: timeout waiting for commit finish!`. The hardware is failing to signal back to the software that a frame has been displayed.

### D. SystemUI & Keyguard Issues
- **Crash Loop:** `com.android.systemui` crashes due to a `NullPointerException` in `StatusBarKeyguardViewManager`. 
- **Cause:** It expects a certain display state or "NotificationShade" view that isn't being initialized because the display hardware hasn't successfully "handshaked" with the framework.

## 3. Bootloader Status

* **Reported Status:** `unlocked: no`
* **Actual Behavior:** Effectively permissive in bootloader fastboot.
* **Fastboot (bootloader mode):**

  * `fastboot flashing unlock` → not supported
  * `fastboot oem unlock` → rejected / invalid
  * No functional unlock mechanism implemented
  * Physical partitions (e.g. `boot_a`, `super`) can be flashed despite reported locked state
* **Fastbootd (userspace fastboot):**

  * Enforces lock state properly
  * Refuses logical partition operations (e.g. resizing `system_a`)
  * Dynamic partition modifications blocked when reported as locked
* **OEM Unlock Support:**

  * `ro.oem_unlock_supported=1`
  * No visible UI toggle
  * No working unlock command
  * Bootloader does not implement standard Android unlock protocol

### Summary

- The device does not implement a functional Android bootloader unlock mechanism.
However, the bootloader-level fastboot interface allows flashing of physical partitions (including `super`) regardless of reported lock state.

- Lock enforcement exists only in fastbootd (userspace), not in bootloader fastboot.

## 4. Attempted Fixes & Workarounds
- **HAL Compatibility:** Mounted `libminijail.so` from VNDK v28 to `/vendor/lib/libminijail_vendor.so` to satisfy old Allwinner HAL dependencies.
- **Graphics Routing:** Set `ro.hardware.egl=mali` and `ro.hardware.gralloc=ares` to force the use of vendor-specific graphics drivers.
- **Provisioning:** Force-set `device_provisioned` and `user_setup_complete` to `1` to bypass the Setup Wizard.
- **Resolution Tuning:** Reset `wm size` to `1280x720` to ensure the software matches the projector's physical panel.

## 5. Conclusion
The device is capable of rendering (proven by the "wall image" when data is unmounted), but the proprietary Allwinner Display Stack is highly brittle. The GSI framework's attempt to take full control of the display (loading wallpapers, UI layers, etc.) causes the HWC to hang or the kernel to trigger a blanking safety mechanism. Permanent success would likely require a custom-built vendor implementation or a "shim" for the SurfaceFlinger-HWC communication.
