---
layout: post
---

# Installing OpenBSD 7.9 on NanoPi R4S – Detailed Guide

This guide provides step‑by‑step instructions for installing OpenBSD 7.9 on the FriendlyElec NanoPi R4S single‑board computer. Since there is no official installation image for this board, we will use the ARM64 `miniroot` image together with a custom device tree blob (DTB) and a compatible U‑Boot bootloader.

---

## Prerequisites

### Hardware
- NanoPi R4S board
- MicroSD card (at least 8 GB, class 10 or better)
- USB‑C power supply (5 V / 3 A)
- **USB‑to‑TTL serial adapter** (3.3 V logic level, **not 5 V**) – required to access the debug UART for installation and troubleshooting

### Software
- A computer running **macOS** (the instructions below are tailored for macOS; Linux users can adapt the `dd` commands accordingly)
- Terminal application (built‑in)

---

## Step 1: Download Required Files

Download the following four files from the official OpenBSD 7.9 mirror. All links are provided below.

| File | Purpose | Download URL |
|------|---------|--------------|
| `miniroot79.img` | Installation kernel and installer | [https://cdn.openbsd.org/pub/OpenBSD/7.9/arm64/miniroot79.img](https://cdn.openbsd.org/pub/OpenBSD/7.9/arm64/miniroot79.img) |
| `rk3399-nanopi-r4s.dtb` | Device tree blob for NanoPi R4S | [https://cdn.openbsd.org/pub/OpenBSD/7.9/packages/aarch64/dtb-6.14.tgz, unpack it and find it under /share/dtb/arm64/rockchip/](https://cdn.openbsd.org/pub/OpenBSD/7.9/packages/aarch64/dtb-6.14.tgz) |
| `u-boot-aarch64-2021.10p14.tgz` | U‑Boot package (contains the bootloader files) | [https://cdn.openbsd.org/pub/OpenBSD/7.9/packages/aarch64/u-boot-aarch64-2021.10p14.tgz](https://cdn.openbsd.org/pub/OpenBSD/7.9/packages/aarch64/u-boot-aarch64-2021.10p14.tgz) |

After downloading the tarball, extract it and locate the `nanopi-r4s-rk3399` directory. Inside you will find two essential files: **`idbloader.img`** and **`u-boot.itb`**.

Optionally, you may also download [`INSTALL.arm64`](https://cdn.openbsd.org/pub/OpenBSD/7.9/arm64/INSTALL.arm64) for reference.

---

## Step 2: Connect the USB‑to‑TTL Serial Adapter (macOS)

The debug UART on the NanoPi R4S is a 3‑pin header (2.54 mm pitch). This connection is **mandatory** because the installation process requires interactive console access.

### Pinout of the Debug UART Header

| Pin | Signal | Connect to |
|-----|--------|------------|
| 1   | **GND** | TTL adapter **GND** |
| 2   | **UART2DBG_TX** (board transmit) | TTL adapter **RXD** |
| 3   | **UART2DBG_RX** (board receive) | TTL adapter **TXD** |

> **Important:**  
> - The logic level is **3.3 V** – make sure your adapter supports 3.3 V and **never** use 5 V adapters.  
> - There is **no VCC** pin on this header; do **not** connect the adapter’s VCC line.

### Serial Connection on macOS

1. **Plug in the USB‑to‑TTL adapter** to your Mac.  
   Check the device name with:

   `ls /dev/tty.*`

You should see something like /dev/tty.usbserial-XXXX or /dev/tty.wchusbserial-XXXX.

2. Open a serial terminal using the built‑in screen command:

`screen /dev/tty.usbserial-XXXX 1500000`

Replace the device name with the one found above. The baud rate is 1,500,000.

Keep this terminal window open – all boot and installation messages will appear here.

## Step 3: Prepare the MicroSD Card

Warning: This process erases all data on the SD card. Back up any important files before proceeding.

### 1. Identify the SD card device on macOS:

`diskutil list`

### 2. Unmount the SD card (do not eject it):

`sudo diskutil unmountDisk /dev/disk2`

(Replace /dev/disk2 with your actual device.)

### 3. Write the miniroot image and U‑Boot to the SD card.

Use the raw device (rdisk) for faster writing:

```
# Write miniroot image
sudo dd if=miniroot79.img of=/dev/rdisk2 bs=1m conv=sync status=progress

# Write idbloader.img at offset 64 (seek=64)
sudo dd if=idbloader.img of=/dev/rdisk2 bs=512 seek=64 conv=sync status=progress

# Write u-boot.itb at offset 16384 (seek=16384)
sudo dd if=u-boot.itb of=/dev/rdisk2 bs=512 seek=16384 conv=sync status=progress
```

### 4. Copy the DTB file to the FAT partition that now appears on the SD card.

After writing, the card will have a FAT partition (usually auto‑mounted under /Volumes/). Navigate to that mount point:

* `cd /Volumes/NAME_OF_PARTITION/`
* `mkdir rockchip`
* `cp /path/to/downloaded/rk3399-nanopi-r4s.dtb rockchip/`

(Replace NAME_OF_PARTITION with the actual volume name, e.g., MINIROOT.)

## Step 4: Boot the Installer

1. Insert the prepared SD card into the NanoPi R4S.
2. Make sure your serial adapter is connected and the screen session is running.
3. Power on the board by connecting the USB‑C cable.

You should immediately see U‑Boot and OpenBSD boot messages in the screen terminal. After a few seconds, the OpenBSD installer prompt will appear.

## Step 5: Run the OpenBSD Installation

The installer is mostly interactive and similar to the standard OpenBSD installation. Pay special attention to the following points:

1. Select the root disk – this is the most critical step.
  * Ideally, the installer will detect your SD card (usually listed as sd0 or similar).
  * If no disk is listed (you may see an error like sdmmc0: can't enable card), try a different SD card brand/model, or consider installing to a USB drive instead. Some users have succeeded with USB installation, but booting from USB may require additional steps.
2. Follow the prompts for hostname, network configuration, choice of installation sets, etc. – these are identical to a standard OpenBSD installation.

## Step 6: First Boot and Post‑Installation

After the installation completes, the system will ask you to reboot. Remove the SD card (or USB drive, if you used one) and power cycle the board. The serial console will show the OpenBSD boot log, and you will eventually see the login prompt.

---

## References

* [OpenBSD 7.9 ARM64 installation guide (official)](https://cdn.openbsd.org/pub/OpenBSD/7.9/arm64/INSTALL.arm64)
* [NanoPi R4S hardware specifications](http://wiki.friendlyelec.com/wiki/index.php/NanoPi_R4S)
