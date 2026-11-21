# Recovery Device Tree for the Infinix Smart 9 HD (X6532C)

OrangeFox / TWRP device tree for the Infinix Smart 9 HD (codenamed "X6532C"), a
MediaTek MT6765 smartphone with A/B + Virtual A/B and no dedicated recovery
partition.

This tree was written from a read-only dump of the device itself, not copied from
any public tree. It is a companion to the LineageOS 19.1 device tree, and both
use the same stock kernel, dtb and kernel modules.

## Device specifications

| Feature     | Specification                              |
| :---------- | :----------------------------------------- |
| Chipset     | MediaTek MT6765 (Helio G50), 12 nm         |
| CPU         | Octa-core (8x Cortex-A53, up to 2.2 GHz)   |
| GPU         | Imagination PowerVR GE8320                 |
| Memory      | 3 / 4 GB LPDDR4X                           |
| Storage     | 64 GB eMMC 5.1, microSDXC up to 1 TB       |
| Display     | 6.7 inch IPS LCD, 720 x 1600 (HD+), 90 Hz  |
| Battery     | 5000 mAh Li-Po, 10 W wired charging        |
| Dimensions  | 165.7 x 77.1 x 8.4 mm, 188 g               |
| Fingerprint | Side-mounted (power button)                |

## Recovery notes

The device has **no `recovery` partition**. The recovery resources are moved into
**`vendor_boot`** via `BOARD_MOVE_RECOVERY_RESOURCES_TO_VENDOR_BOOT := true`. The
kernel stays in `boot` and is never touched.

The resulting image is `vendor_boot.img` (64 MiB) and boots as recovery through
the stock bootloader, so the device keeps its original `boot`, `dtbo` and `vbmeta`
layout.

Kernel modules are loaded from the vendor ramdisk, not from the GKI module list:

```
TW_LOAD_VENDOR_MODULES          := "$(shell cd $(DEVICE_PATH)/recovery/root/lib/modules && echo *.ko)"
TW_LOAD_VENDOR_MODULES_EXCLUDE_GKI := true
TW_LOAD_VENDOR_BOOT_MODULES      := true
```

## Features

- Boots as recovery from `vendor_boot` (both A/B slots).
- **FBE and metadata encryption work** (Trustonic TEE).
- Fastbootd, to flash logical partitions from inside recovery.
- 156 kernel modules from the stock vendor ramdisk.
- fscrypt policy 2 (`TW_USE_FSCRYPT_POLICY := 2`).

## Decryption (FBE)

The device uses the **Trustonic TEE** (`ro.hardware.gatekeeper=trustonic`), so
recovery has to ship the TEE stack as well:

```
recovery/root/vendor/                       43 vendor blobs (Trustonic HAL, mcRegistry, ...)
recovery/root/init.decrypt.rc               init script that starts the keystore/gatekeeper HALs
recovery/root/system/etc/recovery.fstab     FBE + keydirectory flags
```

This is the most fragile part; if any blob or TEE service fails to start, `/data`
cannot be decrypted.

## Build

```bash
source build/envsetup.sh
lunch twrp_X6532C-eng
mka adbd vendorbootimage
```

## Flash

Flash the produced `vendor_boot.img` to the active slot (this device is on
**slot b**):

```bash
fastboot flash vendor_boot_b vendor_boot.img
```

or from a root shell with `dd`:

```bash
dd if=vendor_boot.img of=/dev/block/by-name/vendor_boot_b
```

> Never flash `boot`, `init_boot`, `preloader`, `seccfg`, `nvram`, `nvdata` or
> `persist`. To roll back, flash the stock `vendor_boot_b` back.

`vbmeta` does not need to be reflashed — the active slot's `vbmeta` has flags=3
(verification and hashtree disabled), so the unsigned footer already matches.

## Copyright

```
#
# Copyright (C) 2025 The LineageOS Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
```
