# Device tree (recovery) — Infinix Smart 9 HD (X6532C)

OrangeFox / TWRP device tree for the **Infinix Smart 9 HD (X6532C)**,
MediaTek **MT6765**, Android 12.1, A/B + Virtual A/B.

This device has **no `recovery` partition** — the recovery resources are moved
into **`vendor_boot`** via `BOARD_MOVE_RECOVERY_RESOURCES_TO_VENDOR_BOOT := true`.
The kernel stays in `boot` and is never touched.

## Features

- Boots as recovery from `vendor_boot` (A/B slots).
- **FBE + metadata encryption works** (Trustonic TEE).
- Fastbootd (to flash logical partitions from inside recovery).
- 156 kernel modules from the stock vendor ramdisk.

## Decryption (FBE)

The device uses the **Trustonic TEE** (`ro.hardware.gatekeeper=trustonic`), so
recovery has to ship the TEE stack as well:

```
recovery/root/vendor/                       43 vendor blobs (Trustonic HAL, mcRegistry, ...)
recovery/root/init.decrypt.rc               init script that starts the keystore/gatekeeper HALs
recovery/root/system/etc/recovery.fstab     FBE + keydirectory flags
```

This is the most fragile part; if any blob/TEE service fails to start, `/data`
cannot be decrypted.

## Build

```bash
source build/envsetup.sh
lunch twrp_X6532C-eng
mka adbd vendorbootimage
```

## Flash

The produced image is `vendor_boot.img` (64 MiB). Flash it to the active slot
(this device is on **slot b**):

```bash
fastboot flash vendor_boot_b vendor_boot.img
```

or from a root shell with `dd`:

```bash
dd if=vendor_boot.img of=/dev/block/by-name/vendor_boot_b
```

> Never flash `boot`, `init_boot`, `preloader`, `seccfg`, `nvram`, `nvdata`,
> `persist`. Rollback: flash the stock `vendor_boot_b` back.

`vbmeta` does not need to be reflashed — the active slot's `vbmeta` has flags=3
(verification + hashtree disabled), so the unsigned footer already matches.

## Source

This tree was built from a read-only dump of the device itself, not copied from
any public tree.

---

# Device tree (recovery, ID) — Infinix Smart 9 HD (X6532C)

Device tree **OrangeFox / TWRP** untuk **Infinix Smart 9 HD (X6532C)**,
MediaTek **MT6765**, Android 12.1, A/B + Virtual A/B.

Device ini **tidak punya partisi `recovery`** — resource recovery dipindah ke
**`vendor_boot`** lewat `BOARD_MOVE_RECOVERY_RESOURCES_TO_VENDOR_BOOT := true`.
Kernel tetap di `boot` dan tidak disentuh.

## Fitur

- Boot recovery dari `vendor_boot` (slot A/B).
- **FBE + metadata encryption jalan** (Trustonic TEE).
- Fastbootd.
- 156 modul kernel dari ramdisk vendor stock.

## Dekripsi (FBE)

Device pakai **Trustonic TEE**, jadi recovery harus ikut membawa stack TEE-nya
(`recovery/root/vendor/`, `init.decrypt.rc`, `recovery.fstab`). Ini bagian paling
rawan: kalau salah satu blob/TEE gagal start, `/data` tidak bisa didekripsi.

## Build & flash

```bash
lunch twrp_X6532C-eng
mka adbd vendorbootimage
fastboot flash vendor_boot_b vendor_boot.img   # slot aktif = b
```

Jangan flash `boot`, `init_boot`, `preloader`, `seccfg`, `nvram`, `nvdata`,
`persist`. Rollback: flash balik `vendor_boot_b` stock. `vbmeta` tidak perlu
diflash ulang.
