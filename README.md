# RX 580 2048SP (6FDF) → RX 570 (67DF) Patched VBIOS

Patched VBIOS for the China-market **Radeon RX 580 2048SP 8GB** so it reports as an
**RX 570 (Device ID `67DF`)**, which fixes driver compatibility on macOS
(`6FDF` is not recognized there). On Linux and Windows the card works either way;
the patch is cosmetic on those platforms.

> **Read this first:** flash at your own risk. A bad flash can brick the card.
> Always keep a backup and a recovery path (iGPU/second GPU) before flashing.

## Source card

| Item | Value |
|---|---|
| Card | Radeon RX 580 2048SP 8GB (Polaris 20) |
| Board | D00033 580S A1 GDDR5 256Mx32 8GB 300e/300m |
| BIOS version | 015.050.002.001.000000 |
| Original Device ID | `1002:6FDF` |
| Patched Device ID | `1002:67DF` (Ellesmere RX 470/480/570/580 family) |
| Size | 524288 bytes (512 KB) |
| VRAM | 8 GB GDDR5 |

## Files

| File | Description | SHA256 |
|---|---|---|
| `rx580_backup.rom` | Original unmodified backup (`6FDF`) | `62de0ec74a61800da18bcaf750b6573710c40dd005746ccf1d144e4fc9514241` |
| `rx580_backup_patched.rom` | Patched (`67DF`), checksum-fixed | `6e2bf158d631de0ab2d67408cafaf71183a2d3a70de3ab3bf990b6bd625e7682` |

Verify after download:

```bash
sha256sum *.rom
```

## What the patch changes (nothing else is touched)

1. Offset `0xD4`: `F0 FD E6 0F` → `F0 7D E6 0F`
2. PCI Device ID `02 10 DF 6F` → `02 10 DF 67` (2 occurrences: legacy PCIR + UEFI/GOP PCIR)
3. Legacy checksum byte at `0x21` recalculated (`sum % 0x100 == 0`)

Clocks, voltages, fan curve, and memory timings are **identical** to stock.

## Flashing (Linux)

```bash
sudo amdvbflash -i
sudo amdvbflash -s 0 my_backup.rom        # extra safety dump
sudo amdvbflash -unlockrom 0
sudo amdvbflash -p 0 rx580_backup_patched.rom
sudo reboot
lspci -nn | grep 01:00.0                  # expect [1002:67df]
```

Flash back to stock with the same steps pointing at `rx580_backup.rom`.
Keep the monitor on iGPU/motherboard output while flashing the discrete card.

## Compatibility warning

- Use these files **only** on the same board (`D00033`, 8 GB). Flashing a VBIOS
  from a different board/memory configuration can cause instability or no boot.
- This VBIOS is vendor firmware. Redistributing it may violate the vendor's
  copyright or GitHub's ToS. The safer way to share this mod is the patcher
  script (applies the 3 changes above to your own dump) instead of the ROMs.
