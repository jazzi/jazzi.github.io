---
layout: post
---

My wife has a desktop that used to be for desining work, very powerful at that time, I mean ten years ago, sure the OS was windows. Now it's abondoned though still enough for my daily work, I am trying to find the right operating system accourding to the parts in the box:

* CPU: Intel Xeon E3-1231 @3.4GHz x 8
* Memory: Kinston DDR3 1600 8GiB x 4
* Graphics: NVIDIA Quadro K620
* Motherboard: ASUS Z97-P
* Disk: WD Blue SN570 NVMe SSD PCIe3.0 500GB
* Network card: Realtek 8111GR Gigabit
* Audio: Realtek ALC891

The release of Slackware 15.0 is the reason I bought this NVME SSD, after tried several other Linux distributions, I finally decided to rest with Slackware though encounter some difficulites, hereby is the major problems.

## EFI partition or not?

As my machine start under UEFI mode, GPT table, so before the installation starts, there is a need for hard drive partition, EFI partiion is a MUST have.

```
Partition 1: nvme0n1p1: 2G : EFI System
Partition 2: nvme0n1p2: 50G: Linux File System for root /
Partition 3: nvme0n1p3: 16G: SWAP
Partition 4: nvme0n1p4: 397G:Linux File System for /home
```

The HEX code:
1. EFI System: ef00
2. Linux File System: 8300
3. SWAP: 8200

Then the EFI System partition **NEEDs** to be formatted as **vfat** file system:

`mkfs.vfat -n "EFI System" /dev/nvme0n1p1`

After that you will be ready to type the master command "setup" to install it.

## Nvidia driver

As my card is Quadro K620, the default installed driver *nouveau* needs to be removed in advance, so that's the procedure:

1. slackpkg remove xf86-video-nouveau-blacklist
2. sbopkg -i nvidia-kernel
3. sbopkg -i nvidia-driver

The package **sbopkg** is a tool to download and install from https://slackbuilds.org and it's not there after system installation, you need to grab and install it from https://sbopkg.org. A pre-built package of 0.38.2 is [here](https://github.com/sbopkg/sbopkg/releases/download/0.38.2/sbopkg-0.38.2-noarch-1_wsr.tgz).

**One note, to install Nvidia driver, you need to switch to text mode, logout your desktop in advance.**

Regarding install packages through command *sbopkg*, I personally perfer to use **Queuefiles** which can automatically install another package if it's **required**, in short, it makes job easier. To create the queue file, just type:

`sqg -p <package> # this will build queue file for a single package`

## FCITX Chinese Input method

*fcitx* is already installed if you choose FULL installation but not started yet. To make fcitx5 the default input method, please add these lines to your /etc/environment (or .profile):

```
    GTK_IM_MODULE=fcitx
    QT_IM_MODULE=fcitx
    XMODIFIERS=@im=fcitx
```

Logout or restart and enjoy it.

## Others to do after system installation

1. Create regular user by command `adduser`
2. Configure *sudo* to easy daily work

```
# visudo
(...)
%wheel ALL=(ALL:ALL) ALL
newuser ALL=(ALL:ALL) ALL  # add this new line
# Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" # remove the # to let regular user can run command *installpkg* etc.
```

For the rest you may visit [TuM'Fatig's website](https://www.tumfatig.net/2022/slackware-linux-15-with-fde-on-uefi-laptop/#:~:text=Slackware%20Linux%2015%20with%20FDE%20on%20UEFI%20laptop,be%20entered%20with%20the%20configured%20keyboard%20layout.%20)

## Error: Startx failed after kernel upgrade

Some may use the following commands to upgrade the Slackware system:

```
#slackpkg update
#slackpkg install-new
#slackpkg upgrade-all
```

Everything should works except one thing, the **Elilo**, if above command upgrade kernel, the Elilo will not update its configuration automaticly, thus need you to do it manually before reboot.

1. mount /boot/efi
2. cp /boot/{vmlinux,vmlinuz-generic} /boot/efi/EFI/Slackware
3. /usr/sbin/geninitrd
4. cp /boot/initrd.gz /boot/efi/EFI/Slackware

The above copy dummies could be replaced with command **eliloconf**.

Command **geninitrd** could be replaced with command **mkinitrd**, in order to use right options for command mkinitrd, the following command could be excuted in advance:

`/usr/share/mkinitrd/mkinitrd_command_generator.sh -k 5.15.94 # Kernel version 5.15.94`

In order to ommit kernel upgrading, we can blacklist them by editing /etc/slackpkg/blacklist.

---

Finally hereis my elilo.conf:

```
chooser=simple
delay=1
timeout=1
default=vmlinuz
#
image=vmlinuz
        label=vmlinuz
        initrd=initrd.gz
        read-only
        append="root=/dev/nvme0n1p2 vga=normal ro"
#
# Linux bootable partition config begins
# initrd created with 'mkinitrd -c -k 5.15.94 -f ext4 -r /dev/nvme0n1p2 -m usb-storage:xhci-hcd:xhci-pci:ohci-pci:ehci-pci:uhci-hcd:ehci-hcd:hid:usbhid:i2c-hid:hid_generic:hid-asus:hid-cherry:hid-logitech:hid-logitech-dj:hid-logitech-hidpp:hid-lenovo:hid-microsoft:hid_multitouch:jbd2:mbcache:crc32c_intel:crc32c_generic:ext4 -u -o /boot/initrd.gz'
image=vmlinuz-generic
	initrd=initrd.gz
	label=generic
	read-only
	append="root=/dev/nvme0n1p2 vga=normal ro"
```
