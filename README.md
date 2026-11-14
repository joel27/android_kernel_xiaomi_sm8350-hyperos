> This project is the source code of the Android/HyperOS kernel for Xiaomi SM8350 (Lahaina) devices. The current maintenance focus is on the Xiaomi 11 Pro (mars / M2102K1AC), the Xiaomi 11 Ultra (star / M2102K1G) and the Xiaomi 11 (venus / M2011K2G).
>
> The Android 17 features include the double-tap-to-wake function, Edge compatibility fixes, audio sample-rate fixes, and strict kernel-only installation restrictions (see the release notes). For a complete list of earlier releases, see the release history. The older boot flashing instructions do not apply to the current release process.

# Dynamic Kernel for SM8350

## Download the Latest Release

**Download the latest release (AK3 kernel package and unified fix module).**

This fixed entry point is used to open the download page of the latest official release, which includes the AK3 kernel package, the unified fix module, and related attachments. The specific versions, update contents, compatibility scope, installation and rollback methods, SHA-256 hashes, and verification limits are subject to the release notes on that page.

## Project Purpose

This repository is used to maintain the Linux 5.4.302 arm64 kernel for the Xiaomi SM8350 platform, and provides source code, configuration, build verification, and AK3 kernel package templates for the Android/HyperOS runtime environment of the Xiaomi 11 Pro / 11 Ultra / 11 (venus).

The project follows `WORK_RULES.md`. The release process follows the principles of minimal modification, evidence first, and traceability. Source modification, kernel build, AK3 packaging, temporary boot, and release are independent phases, and each one requires separate verification and authorization.

The current main maintenance branch of the repository is `dynamic`. The default device configuration is `arch/arm64/configs/vendor/star_defconfig`. This configuration has Clang ThinLTO enabled: `CONFIG_LTO_CLANG=y` and `CONFIG_THINLTO=y`.

## Project Structure

| Path | Purpose |
|---|---|
| `Makefile` | The top-level Linux kernel Kbuild entry point, responsible for configuration, build, linking, and version generation. |
| `arch/arm64/` | arm64 architecture startup code, memory management, platform support, and device configuration. |
| `arch/arm64/configs/vendor/star_defconfig` | The default configuration for the star HyperOS/Android 17 target; it is also the current build baseline. |
| `drivers/` | Drivers for input, touch, display, USB, storage, power, vibration, and network devices. |
| `techpack/` | Collection of Qualcomm platform-specific drivers, including Lahaina audio, display, and other SoC-related implementations. |
| `kernel/`, `mm/`, `fs/`, `block/`, `ipc/`, `lib/` | Core subsystems: kernel scheduling, memory, file systems, block devices, inter-process communication, and general libraries. |
| `net/`, `crypto/`, `security/` | Network protocols, cryptographic algorithms, and security mechanisms. |
| `include/` | Internal kernel header files, and the `include/uapi/` UAPI definitions visible to user space. |
| `KernelSU/` | KernelSU-related kernel driver code; the manager APK is not installed with the kernel package. |
| `scripts/dynamic/` | Dynamic project scripts for AK3 generation, kernel package auditing, offline verification, sample-rate checks, and boot repack verification. |
| `scripts/ak3/` | The AK3 fixed template source. The template directory does not keep an `Image`; during packaging, only the newly built and verified kernel `Image` is placed in the package root directory. |
| `Documentation/` | Kernel documentation, feature fixes, test limits, and release notes of this project. |
| `WORK_RULES.md` | Work rules for error detection, code modification, new features, builds, commits, AK3, and release. |

## Main Entry Points and Artifacts

- **Kernel build entry point**: the root `Makefile`.
- **Device configuration entry**: `arch/arm64/configs/vendor/star_defconfig`.
- **arm64 kernel artifact**: `arch/arm64/boot/Image` (when `O=` is used, it is located in `arch/arm64/boot/Image` under the corresponding output directory).
- **AK3 generation entry point**: `scripts/dynamic/package_kernel_only.py`.
- **AK3 installation entry point**: `scripts/ak3/anykernel.sh` and the main AK3 scripts in `tools/`.
- **Offline verification entry points**: `scripts/dynamic/test_kernel_only.py`, `scripts/dynamic/test_ak3_offline.py` and `scripts/dynamic/verify_boot_repack.py`.
- **Audio sample-rate verification entry point**: `scripts/dynamic/test_tdm_sample_rates.py`.

In the build commands, the `ARCH`, `LLVM`, `CROSS_COMPILE` and `O=` parameters must be filled in according to the actual toolchain environment. The following commands only represent the Kbuild targets and do not imply that this repository will install the build environment automatically:

```
make O=<output-directory> ARCH=arm64 vendor/star_defconfig
make O=<output-directory> ARCH=arm64 Image
```

Before official packaging, a source snapshot must be committed, and it must be confirmed that there are no undocumented source changes in the working tree. Windows working trees may have problems caused by case-insensitive paths and reserved names such as `aux.*`; keeping those names produces differences in Git. These differences must be recorded and handled as described in `WORK_RULES.md`, and must not be committed directly as source modifications.

## Main Functional Modules

- Qualcomm Lahaina/SM8350 platform support, as well as boot and power management adaptation for Xiaomi devices.
- `techpack/audio/asoc/lahaina.c`: TDM sample-rate conversion and audio path logic.
- `drivers/input/touchscreen/fts_spi/`: touch drivers, including screen-off gestures and double-tap-to-wake handling.
- Android 17 capabilities, BPF, RCU task tracing, netbpfload, and hrtimer compatibility paths.
- Xiaomi Binder interceptor, reference protection, IPv4/IPv6 network events, and UID filtering.
- Kernel-side extensions such as ReKernel-X, BakaSU (formerly ReSukiSU), SUSFS, and KernelSU; the specific versions and sources are subject to the source code and the release notes.
- AW8697 haptic (vibration) driver, delayed loading of the RAM firmware, waveform sequencing, and existing gain control.
- Configuration capabilities such as `schedutil` scheduling / WALT, ZRAM lz4p, F2FS/EROFS, `seccomp`, and BPF syscalls/JIT.

The modules mentioned above are the parts of the source code directly related to the device functionality and the current baseline version, and they do not imply that this repository contains only these subsystems. Before modifying any module, you must first complete a full read-only analysis and identify the root cause, as described in `WORK_RULES.md`.

## Root Component Source Version

`KernelSU/` is currently synchronized with the official BakaSU repository `main`, commit `373303c525d05c8b47801305f506ef9124dbd6e6`, version number `35218`, reference tag `v4.2.0-rc3`, UAPI `5`. This is development source code; the source and version metadata are recorded in `KernelSU/resukisu-version`.

This repository retains the source code for direct integration, the SELinux 5.4 state locks, and the original multi-manager signature functionality, as well as `KSU_SUKISU` module identifier compatibility. The device defconfig, SUSFS 2.3.0, ReKernel-X, the unified fix module, and the AK3 template keep their existing configurations. The update records and verification limits are detailed in the BakaSU 35218 sync records.

This source code has completed basic temporary real-device verification on the star, including boot, manager access, UAPI 5 compatibility, loading of existing fix modules, and protection against duplicate services; the original operating environment was restored after the tests. The process, known issues, and untested areas can be found in the real-device test and rollback records.

The manager APK is installed by the user and must be compatible with the kernel and the `ksud` UAPI. The published rc3 / 35171 APK still uses UAPI 4 and cannot be used as the companion version for this UAPI 5 update. The basic temporary real-device verification has been completed, and the release `20261008-2155` has been published. Extended standby, full location tracking, the Xiaomi 11 Pro (mars), the Xiaomi 11 (venus), and permanent installation are not included in the test scope according to the release notes. Temporary tests are not considered full acceptance.

## AK3 Kernel Package Rules

- The repository's AK3 package is generated from the `scripts/ak3/` directory. This directory only stores committed template files, not compiled `Image` files.
- Without explicit permission from the user, only the final kernel `Image` built from the template may be replaced; the ramdisk, `patch/`, modules, DTB/DTBO, vbmeta, or any other partition content must not be modified.
- During real packaging, `scripts/dynamic/package_kernel_only.py` adds the newly built `arch/arm64/boot/Image` to the package root directory and verifies the image hash and the kernel version.
- The currently allowed devices are `mars`, `star`, `venus`, `M2102K1AC`, `M2102K1G` and `M2011K2G`. Devices, recoveries, bootloaders, Android versions, or ROMs that have not been verified must not have their own admission criteria added.
- The installation script uses `split_boot` and `flash_boot` to replace only the kernel payload in the existing boot image and keeps the original ramdisk; `patch_vbmeta_flag=0`, `slot_select=active`.
- After the AK3 package is generated, checks for template integrity, device admission, ZIP integrity, image consistency, and offline boot repack must be performed. Packages that do not pass these checks must not be used for temporary boot, flashing, or release.

## Test and Release Limits

- During analysis or testing, only the existing source code, logs, configurations, and artifacts are read; no modification, build, packaging, or device flashing is performed.
- After a code fix, the test scope must include at least the original problem, the affected modules, and the necessary boot/regression checks; the test results, the untested items, and the remaining risks must be recorded.
- Temporary real-device tests with `fastboot boot` should be given priority, with rollback as the fallback; permanent flashing, rebooting, slot switching, and partition modification each require separate authorization.
- The release tag reuses the AK3 time format `<YYYYMMDD-HHmm>`, for example `20261001-0213`. A release must include the source commit, the kernel image, the AK3 package, the hashes, the build manifest, the offline verification results, and the live-test limits.
- Changes to documentation, scripts, and rules files can be committed separately without regenerating the kernel image; however, an official kernel release still requires full build and release verification.

## How to Submit a Patch to the Android Common Kernel

- **Preferred approach**: First make the changes in upstream Linux, then backport them to a stable release when appropriate. The corresponding patch will be merged automatically into the Android common kernel. If the patch already exists in upstream Linux, an adaptation that meets the requirements below should be submitted.
- **Alternative approach**: Develop the patch outside the Linux repository. From the perspective of the upstream Linux project, unless the patch fixes a specific Android problem, it is generally not accepted; coordinate with the developers by contacting `kernel-team@android.com` before submitting, and meet the patch requirements below.

### Common Kernel Patch Requirements

- All patches must comply with the Linux kernel coding standards and pass `scripts/checkpatch.pl`.
- The patch must not break the `gki_defconfig` or `allmodconfig` build on the ARM, ARM64, x86, or x86_64 architectures (see https://source.android.com/setup/build/building-kernels).
- If the patch is not merged from an upstream branch, the subject must specify the patch type: `UPSTREAM:`, `BACKPORT:`, `FROMGIT:`, `FROMLIST:` or `ANDROID:`.
- All patches must include a `Change-Id:` tag (see https://gerrit-review.googlesource.com/Documentation/user-changeid.html).
- If an Android bug has already been assigned, it must be included in a `Bug:` tag.
- Patches must include the name of the author and of the submitter in a `Signed-off-by:` tag.

The following requirements apply depending on the patch type.

#### Backports from Linux mainline: `UPSTREAM:`, `BACKPORT:`

- If the patch is a cherry-pick from Linux mainline and there are no changes:
  - Use the `UPSTREAM:` subject;
  - Add the `(cherry-picked from ...)` upstream commit information.
- If the patch requires changes relative to the upstream version: use `BACKPORT:`, keep the same tags as `UPSTREAM:` and the `(cherry-picked from ...)` line, and add a note below it explaining the changes.

Example:

```
UPSTREAM: important patch from upstream

This is the detailed description of the important patch

Signed-off-by: Fred Jones <fred.jones@foo.org>

Bug: 135791357
Change-Id: I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
(cherry-picked from c31e73121f4c1ec41143423ac6ce3ce6dafdcec1)
Signed-off-by: Joe Smith <joe.smith@foo.org>
```

If there is a backport conflict, it must be recorded after the source commit line, for example:

```
BACKPORT: important patch from upstream

[ Resolved minor conflict in drivers/foo/bar.c ]
```

#### Backports from other sources: `FROMGIT:`, `FROMLIST:`

- When a patch has been merged into a maintainer's branch but has not yet been added to Linux mainline, the subject uses `FROMGIT:` and records `(cherry picked from commit <sha1> <repo> <branch>)`. The source must be a stable, non-rebasing maintenance branch; do not use `linux-next`.
- If the patch still needs modifications, use `BACKPORT: FROMGIT:`.
- When a patch has been posted on LKML but has not yet entered any maintainer branch, the subject uses `FROMLIST:` and adds a `Link:` tag pointing to the `lore.kernel.org` submission page.
- If the patch still needs modifications, use `BACKPORT: FROMLIST:`.

Examples:

```
FROMGIT: important patch from upstream

Bug: 135791357
(cherry picked from commit 878a2fd9de10b03d11d2f622250285c7e63deace
 https://git.kernel.org/pub/scm/linux/kernel/git/foo/bar.git test-branch)
Change-Id: I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
Signed-off-by: Joe Smith <joe.smith@foo.org>
```

```
FROMLIST: important patch from upstream

Bug: 135791357
Link: https://lore.kernel.org/lkml/20190619171517.GA17557@someone.com/
Change-Id: I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
Signed-off-by: Joe Smith <joe.smith@foo.org>
```

#### Android-specific patches: `ANDROID:`

- When fixing bugs in Android-specific code, the subject uses `ANDROID:` and adds a reference to the issue in a `Fixes:` tag.
- When adding new Android-specific features, the subject uses `ANDROID:` and adds the Android bug in a `Bug:` tag.

Example:

```
ANDROID: fix android-specific bug in foobar.c

This is the detailed description of the important fix

Fixes: 1234abcd2468 ("foobar: add cool feature")
Change-Id: I4caaaa566ea080fa148c5e768bb1a0b6f7201c01
Signed-off-by: Joe Smith <joe.smith@foo.org>
```

## HHG Device Vibration Driver

The following content refers to instructions retained from the original repository for integrating the AKSys gamepad USB adapter driver. This driver is not part of the current packaging process of the Xiaomi AK3 devices; confirm that the target kernel requires this driver before using it.

### Integrate the driver into the kernel source tree

1. Copy `${this_project}/drivers/hid/hid-aksys.c` to `${your_kernel_root}/drivers/hid/`.

2. Compare and merge `${this_project}/drivers/hid/hid-ids.h` and `${your_kernel_root}/drivers/hid/hid-ids.h`. Add the following at the end of the file:

   ```
   #define USB_VENDER_ID_QUALCOMM  0x0a12
   #define USB_VENDER_ID_TEMP_HHG_AKSY 0x1234
   #define USB_PRODUCT_ID_AKSYS_HHG  0x1000
   ```

3. Compare and merge `${this_project}/drivers/hid/Kconfig` and `${your_kernel_root}/drivers/hid/Kconfig`. Add the following at the end of the file:

   ```
   config HID_AKSYS_QRD
       tristate "AKSys gamepad USB adapter support"
       depends on HID
       ---help---
       Support for AKSys gamepad USB adapter

   config AKSYS_QRD_FF
       bool "AKSys gamepad USB adapter force feedback support"
       depends on HID_AKSYS_QRD
       select INPUT_FF_MEMLESS
       ---help---
       Say Y here if you have a AKSys gamepad USB adapter and want to
       enable force feedback support for it.
   ```

4. Compare and merge `${this_project}/drivers/hid/Makefile` and `${your_kernel_root}/drivers/hid/Makefile`. Add the following at the end of the file:

   ```
   obj-$(CONFIG_HID_AKSYS_QRD) += hid-aksys.o
   ```

5. Modify the default configuration file of the target kernel and add the following:

   ```
   CONFIG_HID_AKSYS_QRD=m
   CONFIG_AKSYS_QRD_FF=y
   ```

Any source code merge, configuration change, and build action must comply with the phase limits of `WORK_RULES.md`; the merge operations above must not be executed directly before the analysis of the problem is complete and the user has confirmed.
