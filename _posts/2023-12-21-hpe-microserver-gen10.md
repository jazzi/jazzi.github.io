---
layout: post
---

Finally bought a second hand HPE MicroServer Gen10, it's quite old but I heard it's very stable, anyway here is the profile:

* CPU: Opteron X3216 1.6G
* GPU: Integradted AMD Radeon 7
* RAM: ECC DDR4 UDIMM 8GB, could upgrade to 16G * 2
* NIC: Broadcom 5720 Gigabit Ethernet * 2
* SATA: LFF/SFF SATA 6Gb/s * 4 + SSD SATA * 1
* PCIe: PCIe3 x 8 and PCIe3 x 4
* USB: USB 2.0 * 3 and USB 3.0 * 3
* VIDEO PORT: VGA * 1 plus DP * 2
* POWER: 200W

It's quite compact and elegant, the shortage is you need to buy an extra SSD SATA cable and a 4-Pin to SATA power cable or the default P3 cable doesn't work directly.

## Upgrade BIOS

Prepare a USB stick with FAT32 filesystem and unzip the [BIOS firmware from HPE](https://support.hpe.com/hpesc/public/swd/detail?swItemId=MTX_4d74284db6f44592b613b53496) to it.

Don't install any hard drive in advance.

1. Plug Keyboard and USB stick to USB 2.0 port, as well as the monitor and power cord.
2. Power on and press F11
3. Select **UEFI: Built-in EFI Shell**
4. Type `fs0:` to enter USB drive, drive number mostly is 0 if no other dirve installed, try `map -r` to look for the right one
4. Type `cd System_BIOS_v_ZA10A360_for_MicroServer_Gen10_UEFI/EFI` to enter subdirectory of EFI
5. Type `flashbios.nsh` to start the flashing
6. Type `reset` to reboot it after completely flash
