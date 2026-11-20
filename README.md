# Device tree (recovery) — Infinix Smart 9 HD (X6532C)

Device tree **OrangeFox / TWRP** untuk **Infinix Smart 9 HD (X6532C)**,
MediaTek **MT6765**, Android 12.1, A/B + Virtual A/B.

Recovery di device ini **tidak punya partisi `recovery`** — resource recovery
dipindah ke **`vendor_boot`** lewat
`BOARD_MOVE_RECOVERY_RESOURCES_TO_VENDOR_BOOT := true`. Kernel tetap di `boot`
dan tidak disentuh.

## Fitur

- Boot sebagai recovery dari `vendor_boot` (slot A/B).
- **FBE + metadata encryption bekerja** (Trustonic TEE).
- Fastbootd (untuk flash logical partition dari dalam recovery).
- 156 modul kernel dari ramdisk vendor stock.

## Dekripsi (FBE)

Device ini pakai **Trustonic TEE** (`ro.hardware.gatekeeper=trustonic`), jadi
recovery harus ikut membawa stack TEE-nya. Isinya:

```
recovery/root/vendor/        43 blob vendor (Trustonic HAL, mcRegistry, dll)
recovery/root/init.decrypt.rc  script init untuk menyalakan HAL keystore/gatekeeper
recovery/root/system/etc/recovery.fstab   flag FBE + keydirectory
```

Ini bagian paling rawan; kalau salah satu blob/TEE gagal start, `/data` tidak
bisa didekripsi.

## Build

```bash
source build/envsetup.sh
lunch twrp_X6532C-eng
mka adbd vendorbootimage
```

## Flash

Image yang dihasilkan: `vendor_boot.img` (64 MiB). Flash ke slot aktif
(device ini aktif di **slot b**):

```bash
fastboot flash vendor_boot_b vendor_boot.img
```

atau dari root lewat `dd`:

```bash
dd if=vendor_boot.img of=/dev/block/by-name/vendor_boot_b
```

> Jangan flash `boot`, `init_boot`, `preloader`, `seccfg`, `nvram`, `nvdata`,
> `persist`. Rollback: flash balik `vendor_boot_b` stock.

`vbmeta` tidak perlu diflash ulang — slot aktif `vbmeta` flags=3
(verifikasi + hashtree mati), jadi footer unsigned sudah cocok.

## Sumber

Tree ini dibangun dari dump read-only milik device sendiri, bukan disalin dari
tree publik. Repository kerja yang memuat dump + workflow build:
`rynekoo/X6532C-orangefox`.
