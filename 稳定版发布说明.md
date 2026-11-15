# Stable Release Notes

This document records the changes, component versions, feature status, test results, and known limits of each stable release of the Dynamic kernel.

## Naming and Versioning Rules

- Starting with this release branch, the official kernel name is: `kernel-version-Dynamic-g<first 7 characters of the source commit>`.
- Example: `5.4.302-Dynamic-g1a2b3c4`.
- The stable version number is expressed by the Git tag, the Release title, and the build manifest, and is no longer written to `uname -r`; the Release attachment name appends the commit time of the corresponding source code (`YYYYMMDD-HHMM`).
- `g<source commit>` must use the first 7 characters of the commit the release tag points to. The kernel must be rebuilt every time the source commit changes.
- Before an official build, run `scripts/set-dynamic-version.sh <path to .config>`. The script writes `-Dynamic-g<first 7 characters of the commit>` and disables `CONFIG_LOCALVERSION_AUTO`, so that the commit hash is not appended twice.
- Stable releases use semantic versioning: incompatible or major updates raise the major version, new features raise the minor version, and compatibility fixes raise the patch version.
- Every stable release gets a Git tag with a description; the tag, the release notes, the kernel name, and the build artifacts must point to the same source code.
- Temporary test builds may append a test identifier to the end of the name, but must not be published permanently as stable releases.

### Naming used by this repository's builds (star, `HOS4-A17`)

- The official kernel name is `5.4.302-Tech-g<first 7 characters of the source commit>` (for example `5.4.302-Tech-gaf8bd83`). `-Tech` comes from `CONFIG_LOCALVERSION` in the defconfig, and the build workflow fixes the commit suffix, so the name never carries `-dirty`.
- Package files are named `Kernel-HyperOS-TechMaster-11U-<mode>-<YYYYMMDD-HHMM>.zip`, where the time is the UTC commit time of the source code.
- Releases are published in this repository by the build workflow and tagged `Tech-<YYYYMMDD-HHMM>-g<commit>`.
- The same rule applies: the tag, the release notes, the kernel name, and the build artifacts must point to the same source commit.

## HOS4-A17 baseline: Xiaomi 11 Ultra (star)

This is the current baseline of this repository. The first tagged release is created by the build workflow (`root_mode = All-Release`).

### Target device and base version

- Device: Xiaomi 11 Ultra / `star` / M2102K1G. The packages are built for `star`. The installer does not check the device model, so they must only be flashed on that device.
- Other devices of the project: Xiaomi 11 Pro (`mars` / M2102K1AC) and Xiaomi 11 (`venus` / M2011K2G). These packages do not cover them.
- Linux kernel: 5.4.302.
- Compiler: ZyCromerZ Clang 16.0.6 (release `16.0.6-20260807`), built with `LLVM=1` and `LLVM_IAS=1`, on an Ubuntu 24.04 runner with ccache.
- Maintenance branch: `HOS4-A17`.
- Default configuration: `arch/arm64/configs/vendor/star_defconfig`.

### Components

- BakaSU: always the latest official version. The exact version is recorded in each build (the latest verified build used `v4.2.0-rc3-548d1dda`, version code 35259).
- SUSFS: `v2.4.0` and later, with the ReSukiSU/BakaSU inline hook path. The enabled options are `SUS_PATH`, `SUS_MOUNT`, `SUS_KSTAT`, `SPOOF_UNAME`, `OPEN_REDIRECT`, `SUS_MAP`, `HIDE_KSU_SUSFS_SYMBOLS`, `SPOOF_CMDLINE_OR_BOOTCONFIG` and `ENABLE_LOG`. `PRIORITY_UID_FILTER` (new in 2.4.0) is not enabled.
- NoMount: integrated at build time (the latest verified build used `v2.2.0-108-g6298d58`).
- ReKernel-X: kept from the original source tree.
- LTO/CFI: disabled in lite builds (`CONFIG_LTO_NONE`), which is the default of the workflow.

### Build modes

| Mode | Contents |
|---|---|
| `KSU-Susfs` | BakaSU with SUSFS (inline hooks). |
| `KSU` | BakaSU without SUSFS, with manual hooks (`KSU_MANUAL_HOOK`). |
| `Vanilla` | No KernelSU and no SUSFS. |

NoMount is independent of the mode and can be enabled in any of them.

### Changes in this baseline

- SUSFS updated from 2.3.0 to 2.4.0: `fs/susfs.c`, `include/linux/susfs.h` and `include/linux/susfs_def.h` were replaced with the 2.4.0 versions.
- `fs/statfs.c`: added `statfs_by_dentry_wrapper` and `calculate_f_flags_wrapper`, and replaced the `vfs_statfs` hook with the 2.4.0 logic. The previous hook interpreted the return value of `susfs_sus_kstat_spoof_vfs_statfs` the opposite way to 2.4.0.
- `fs/super.c`: added the 2.4.0 hook that gives KSU mounts their own anonymous device numbers.
- `fs/namespace.c`: added the 2.4.0 fast paths for non-sus mounts, and `__lookup_mnt` also covers `zygote_next` processes. The broader check for umounted processes was kept on purpose.
- Hooks in `fs/namei.c`, `fs/readdir.c`, `fs/stat.c`, `fs/proc/fd.c` and `fs/notify/fdinfo.c` keep the source tree's own implementation, which implements the same logic as the 2.4.0 patch.
- Builds are produced by the workflow *Build Xiaomi11U (HyperOS Modern)*: the kernel version is fixed without `-dirty`, the AnyKernel3 installer detects KernelSU, SUSFS and NoMount in the image it flashes, and every build writes a build report.

### Verification

- The build completes without errors or warnings, and the image contains SUSFS `v2.4.0`.
- The installer detection (embedded kernel configuration and string search) was run on the real `Image` and matches the `.config` of the build.
- An earlier build of this branch (SUSFS 2.3.0) was flashed on the device, and the BakaSU manager reported the driver as working.
- Real-device validation of the SUSFS 2.4.0 build has not been recorded yet.

### Known limits

- The 2.4.0 hooks were checked by source review, signature and variable consistency checks, and the build. They have not been exercised by on-device tests yet.
- Real-device validation of the `KSU` and `Vanilla` modes has not been recorded yet. The `KSU` mode relies on the manual hooks of the source tree.
- The matching `susfs4ksu` module and `ksu_susfs` binary must be the same version as the kernel's SUSFS.
- Lite builds do not use LTO/CFI.
- The package does not check the device model.

## Release history inherited from the original project (Xiaomi 11 Pro / mars)

The sections below are the release history of the original Dynamic project for the Xiaomi 11 Pro (`mars`). They are kept for reference and describe only those releases (including their own compiler, branches, and scripts); they do not describe the `star` builds above.

## v1.0.0 (2026-09-17)

### Target device and base version

- Device: Xiaomi 11 Pro / mars / M2102K1AC.
- Linux kernel: 5.4.302.
- Compiler: Ubuntu Clang 17.0.6, LLD 17.0.6.
- Historical release branch: `codex/resukisu-susfs-5.4`; the current maintenance branch has been renamed to `mi11pro/Dynamic-kernel`.
- Release commit: the commit that the Git tag `leeca-kernel-v1.0.0` points to.

### Changes in this version

- Compatibly merged from Linux 5.4.289 up to 5.4.302 in sequence, and added the stable-patch dependencies that were missing in the Xiaomi SM8350 vendor tree.
- Updated the ReSukiSU kernel driver to `v4.2.0-rc2-0b5cffd9@ReSukiSU`; the driver version code is 35149.
- Integrated SUSFS `v2.3.0`, using the ReSukiSU/SUSFS inline hook path.
- Integrated ReKernel-X v1.5; NoActive can connect to the kernel through Generic Netlink.
- Integrated LZ4P and set it as the default ZRAM compression algorithm; the ZRAM size is configured by Android according to the physical memory of the device (12 GiB on the test device).
- Set the default CPU governor to `schedutil`, keeping WALT scheduler load tracking.

### Fixes in this version

- **Black screen and no wake-up after deep sleep:** fixed the missing hrtimer CPU hotplug initialization in the 5.4.290 stable patch. When a CPU comes back online, states such as `active_bases`, `hres_active` and `expires_next` are cleaned again and the expiry lock is initialized, so that leftover timer-base state does not corrupt later suspend/resume.
- **ZRAM flag error:** the compression priority mask and priority value are now converted to `unsigned long` before shifting, which avoids a 32-bit shift overflow on 64-bit flags.
- **ZRAM reset undefined behavior:** removed useless code that read the compressor array with an uninitialized `prio`.
- **DWC3 sleeping in atomic context:** removed the sleeping wait done while holding the gadget spinlock, which avoids `scheduling while atomic`.
- **DWC3 reset wait too long:** changed the soft-reset polling to 10 bounded millisecond-level retries, which avoids a block of about two seconds on abnormal hardware states.
- Added the 5.4-compatible interfaces involved in the stable merge for seal, pidfd, scheduling, SDHCI, Qualcomm GPIO, USB, networking, and others.
- Fixed the USB boot disconnection and the abnormal ZRAM runtime path exposed by the Clang build.

### Current features and versions

- ReSukiSU: `v4.2.0-rc2-0b5cffd9@ReSukiSU`.
- SUSFS: `v2.3.0`.
- ReKernel-X: v1.5; NoActive shows `Netlink · Connected`.
- ZRAM: supports `lz4`, `lz4p` and `zstd`, with `lz4p` as the default.
- CPU scheduling: `schedutil + WALT`.
- Root and hiding: ReSukiSU, SUSFS inline hook.
- Device features: Wi-Fi, cellular network, USB/ADB, camera, audio, sensors, the primary user, and the dual-app user were all included in the real-device checks.

### Real-device verification

- All candidate versions were first booted temporarily with `fastboot boot`; nothing was written to the boot partition during the locating phase.
- Both the primary user 0 and the dual-app user 999 were `RUNNING_UNLOCKED`, and the credential-encrypted data and regular apps were accessible.
- The NoActive interface confirmed that the ReKernel-X Netlink was connected; ReSukiSU privilege elevation worked normally.
- ZRAM was 12 GiB with `lz4p` selected; all three CPU policies were `schedutil`.
- The camera service recognized 10 devices, and the sensor service recognized 73 hardware sensors; the audio and network services were normal.
- With USB disconnected, three rounds of deep sleep of about 3, 5 and 10 minutes were completed, and the device woke up normally every time.
- The same temporary boot lasted about 28 minutes without the boot ID changing; deep suspend succeeded 850 times in total.
- `failed_resume`, `failed_resume_early` and `failed_resume_noirq` were all 0.
- dmesg and the crash buffer showed no panic, Oops, BUG, watchdog, hung task, RCU stall, lockup, or `scheduling while atomic`.

### Known limits

- This long standby verification lasted about 28 minutes and cannot replace several days of real use or all combinations of third-party apps.
- The suspend failure counter of the test device contains `alarmtimer.0.auto / -16 (EBUSY)`. This is an alarm event that makes the suspend phase return device-busy temporarily; the resume failure counter is 0, and the system can retry and wake up normally afterwards.
- The Synthetic Password of the current user data is bound differently from the historical boot chain. The official boot uses the ramdisk that was verified to unlock the current data normally, and replaces only the kernel payload.

### Release requirements

- v1.0.0 is a historical version from before the rename, and its build artifacts keep the name `5.4.302-Leeca-g<first 12 characters of the tag commit>-v1.0.0`; later versions must follow the new Dynamic/7-character-commit rule.
- Before release, check the SHA-256 of the Image and the boot image, and record them in the local stable-release manifest.
- Permanent flashing is allowed only after the temporary boot, decryption, basic function, and deep sleep tests have all passed.

## v1.0.1 (2026-09-17)

### Target device and build information

- Device: Xiaomi 11 Pro / `mars` / M2102K1AC.
- Linux kernel: 5.4.302.
- Maintenance branch: `mi11pro/Dynamic-kernel`.
- Official name: `5.4.302-Dynamic-g<first 7 characters of the release commit>-v1.0.1`.
- Compiler: Ubuntu Clang 17.0.6, LLD 17.0.6.

### Changes and fixes in this version

- Disabled `CONFIG_DEBUG_FS`, so that the debugfs debug interface and its runtime probing surface are no longer exposed to user space.
- Disabled `CONFIG_PAGE_OWNER`, which stops maintaining allocation-source tracking metadata for every page, reducing debugging traces and the corresponding memory and performance overhead.
- Kept `CONFIG_USER_NS=y`, so that disabling user namespaces does not affect Android features or apps that depend on it.
- Modified `star_defconfig`, `lahaina-qgki_defconfig` and `lahaina_QGKI.config` together, so that no configuration merge path re-enables the debugging options above.
- Fixed the `Miscellaneous Check(9)` detection item of Chunqiu Native Check-Eros 4.5.6.
- Updated the AnyKernel3 release template: it limits the target device to the Xiaomi 11 Pro (`mars`), and checks that the kernel Image exists in the package before flashing.
- Fixed `KernelSU/kernel/Kconfig` to LF line endings, so that source code obtained from a Git archive or a fresh checkout can generate the configuration directly on Linux.

### Features and component versions

- ReSukiSU: `v4.2.0-rc2-0b5cffd9@ReSukiSU`.
- SUSFS: `v2.3.0`.
- ReKernel-X: v1.5, with NoActive Generic Netlink connection support.
- ZRAM: supports `lz4`, `lz4p` and `zstd`, with `lz4p` as the default; the size is configured by Android according to the device memory.
- CPU scheduling: `schedutil + WALT`.
- Linux: 5.4.302, including the hrtimer deep sleep, ZRAM and DWC3 stability fixes of v1.0.0.

### Real-device verification record

- The candidate was booted temporarily with `fastboot boot`; nothing was written to the boot partition during testing.
- At runtime it was confirmed that `DEBUG_FS` and `PAGE_OWNER` are disabled, `USER_NS` stays enabled, and debugfs is not mounted.
- `Miscellaneous Check(9)` disappeared from the Chunqiu abnormality list; the `Tampered kernel` item and the USB debugging notice that are still displayed are not targets of this fix.
- The primary user and the dual-app user both unlocked normally, SELinux was Enforcing, and the ReSukiSU root and the NoActive/ReKernel-X connection were normal.
- The 12 GiB ZRAM used `lz4p` normally; a separate zram device completed 10 rounds of data read-back verification and a swap lifecycle test.
- All three CPU policies were `schedutil`; the network, camera, and sensor checks were normal, the camera service recognized 10 devices, and the sensor service recognized 73 hardware sensors.
- The boot ID did not change during the test; dmesg and the crash buffer showed no panic, Oops, BUG, watchdog, hung task, RCU stall, or lockup.

### Known limits and flashing method

- This Check(9) candidate was verified with USB connected, and the long deep-sleep test with USB disconnected that was completed for v1.0.0 was not repeated separately.
- With `DEBUG_FS` disabled, kernel debugging, tracing, and some development tools that depend on debugfs are unavailable; with `PAGE_OWNER` disabled, page allocation-source diagnosis of memory problems is not possible. Neither is a feature of normal daily use.
- The GitHub Release provides an AnyKernel3 flashable package. The package replaces only the kernel in the boot of the current slot and keeps the existing ramdisk; you should still back up the boot image yourself before flashing.

## v1.0.2 (2026-09-20)

Adapted to HyperOS 4 `OS4.0.0.25.XKACNXM` / Android 17 for the Xiaomi 11 Pro (mars). Based on v1.0.1 and the verified test7, it ports the BPF / Tasks Trace RCU compatibility capabilities, restricts the uname version handling of netbpfload, completes the AW8697 LED interface and waveform sequence cleanup, keeps the original gain path, and disables the firmware automatic fallback wait. The hrtimer fixes and the existing Root, SUSFS, ReKernel-X and ZRAM fixes remain as the baseline.

For the complete changes, component versions, adaptation scope, known limits, original verification evidence, and rebuild method, see the [detailed v1.0.2 notes (Chinese)](Documentation/dynamic-v1.0.2-zh.md). The official name is `5.4.302-Dynamic-g<first 7 characters of the tag commit>`; official builds use `scripts/build-dynamic-mars-a17.sh` and `vendor/mars_hyperos4_a17_defconfig`, and release packaging uses `scripts/package-dynamic-release.sh`. No other device or ROM has been announced as tested for compatibility.
