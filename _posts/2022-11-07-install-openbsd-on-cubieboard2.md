---
layout: post
---

What I have in hand:

1. OpenBSD 7.2
2. Cubieboard2

# OS installation

There are two ways to install the system:

1. Plug in monitor and keyboard
2. Connect Cubieboard2 to PC by TTL-USB serial cable, 3.5v

I bought a serial cable and solution 2 didn't work, don't know bad cable or what, anyway I plug on keyboard and monitor, it brings out the text on screen, but there is a trick, the console won't work in that way only, you need to set tty to framebuffer as below.

```bash
>> OpenBSD/armv7 BOOTARM 1.14
boot> set tty fb0
switching console to fb0
```

*Interesting part is OpenBSD 7.1 didn't work, luckily version 7.2 released soon.*

*Another notice is installing software sets through USB stick caused a problem, after installation finished, the reboot didn't work. So I used **http** instead of **local**, that's giving up USB stick.*

The rest is nothing special, just follow the [instructions on OpenBSD](https://ftp.openbsd.org/pub/OpenBSD/7.2/armv7/INSTALL.armv7).

```bash
Location of sets = http
HTTP proxy URL = mirrors.bfsu.edu.cn
HTTP Server = mirrors.bfsu.edu.cn
Server directory = OpenBSD/7.2/armv7
What timezone are you in = Asia/Shanghai
```

## Format new hard drive

My drive was formatted as filesystem ext4 and used under Armbian Linux, now moved to OpenBSD, both partition label and filesystem do not work any more, so need to destroy and create new MBR and FFS as follows:

```bash
#fdisk -i sd0
#disklabel -E sd0
#newfs sd0i (given new partition lable is "i")
```

## Set up NFS service

The trick is, /etc/exports file has different options between Linux and OpenBSD, you can not just copy these Linux options to here, for example, option -rw is not permitted on OpenBSD as it is the default setting, so be careful to read the manual page of [/etc/exports](https://man.openbsd.org/exports).

Here is my exports file:

```bash
/mnt/hdd	-alldirs -network=192.168.0 -mask=255.255.255.0
```

Just follow [Using NFS on FAQ](https://www.openbsd.org/faq/faq6.html#NFS) and everything should be fine.

Above is server part, below is client part.

* MacOS - it needs a special option for mount, that's **resvport**, I put a line into my */Users/robin/.zshrc* file. 

```bash
# Alias to mount NFS server on Cubieboard2
alias nfs="sudo mount -t nfs -o resvport,nolocks,locallocks,soft,async,wsize=32768,rsize=32768 192.168.0.231:/mnt/hdd /Users/robin/nfs"
alias unfs="sudo umount /Users/robin/nfs"
```

* Linux - nothing special, just put the mount into /etc/fstab

The NFS performance is somewhat connected to option *wsize* and *rsize*, 32768/1024=32k, you can tunning it by following this [Optimizing NFS Performance guide](https://nfs.sourceforge.net/nfs-howto/ar01s05.html) and find your best number.

## Troubleshooting

1. I/O Errors - add option *nolock*
2. mount_nfs: can't mount with remote locks when server is not running rpc.statd: RPC prog. not avail - add the following into */etc/rc.local*

```bash
#!/bin/csh
#
# MacOS needs rpc.statd
rpc.statd
```

## Reference resource

* [Preparing microSD on MacOS](https://www.tumfatig.net/2018/running-openbsd-on-raspberry-pi-3/)
* [OpenBSD/armv7 on the CubieBoard2](https://www.cambus.net/openbsd-armv7-on-the-cubieboard2/)
* [Installing OpenBSD on Raspberry Pi 3](https://dev.to/spacial/installing-openbsd-7-on-raspberry-pi-3-1f98)

## dmesg and hw.sensors

```bash
# dmesg
OpenBSD 7.2 (GENERIC) #71: Thu Sep 29 11:47:02 MDT 2022
    deraadt@armv7.openbsd.org:/usr/src/sys/arch/armv7/compile/GENERIC
real mem  = 954286080 (910MB)
avail mem = 926662656 (883MB)
random: good seed from bootblocks
mainbus0 at root: Cubietech Cubieboard2
cpu0 at mainbus0 mpidr 0: ARM Cortex-A7 r0p4
cpu0: 32KB 32b/line 2-way L1 VIPT I-cache, 32KB 64b/line 4-way L1 D-cache
cpu0: 256KB 64b/line 8-way L2 cache
cortex0 at mainbus0
psci0 at mainbus0: PSCI 0.0
sxiccmu0 at mainbus0
agtimer0 at mainbus0: 24000 kHz
simplebus0 at mainbus0: "soc"
sxiccmu1 at simplebus0
sxipio0 at simplebus0: 175 pins
sxirtc0 at simplebus0
sxisid0 at simplebus0
ampintc0 at simplebus0 nirq 160, ncpu 2: "interrupt-controller"
"system-control" at simplebus0 not configured
"interrupt-controller" at simplebus0 not configured
"dma-controller" at simplebus0 not configured
"lcd-controller" at simplebus0 not configured
"lcd-controller" at simplebus0 not configured
"video-codec" at simplebus0 not configured
sximmc0 at simplebus0
sdmmc0 at sximmc0: 4-bit, sd high-speed, mmc high-speed, dma
"usb" at simplebus0 not configured
"phy" at simplebus0 not configured
ehci0 at simplebus0
usb0 at ehci0: USB revision 2.0
uhub0 at usb0 configuration 1 interface 0 "Generic EHCI root hub" rev 2.00/1.00 addr 1
ohci0 at simplebus0: version 1.0
"crypto-engine" at simplebus0 not configured
"hdmi" at simplebus0 not configured
sxiahci0 at simplebus0: AHCI 1.1
sxiahci0: port 0: 3.0Gb/s
scsibus0 at sxiahci0: 32 targets
sd0 at scsibus0 targ 0 lun 0: <ATA, ST1000LM024 HN-M, 2BA3> naa.50004cf20fb99f7f
sd0: 953869MB, 512 bytes/sector, 1953525168 sectors
ehci1 at simplebus0
usb1 at ehci1: USB revision 2.0
uhub1 at usb1 configuration 1 interface 0 "Generic EHCI root hub" rev 2.00/1.00 addr 1
ohci1 at simplebus0: version 1.0
"timer" at simplebus0 not configured
sxidog0 at simplebus0
"ir" at simplebus0 not configured
"codec" at simplebus0 not configured
sxits0 at simplebus0
com0 at simplebus0: dw16550
com0: console
sxitwi0 at simplebus0
iic0 at sxitwi0
axppmic0 at iic0 addr 0x34: AXP209
sxitwi1 at simplebus0
iic1 at sxitwi1
"gpu" at simplebus0 not configured
dwge0 at simplebus0: rev 0x00, address 02:90:09:c2:25:2e
rlphy0 at dwge0 phy 1: RTL8201L 10/100 PHY, rev. 1
"hstimer" at simplebus0 not configured
"display-frontend" at simplebus0 not configured
"display-frontend" at simplebus0 not configured
"display-backend" at simplebus0 not configured
"display-backend" at simplebus0 not configured
gpio0 at sxipio0: 32 pins
gpio1 at sxipio0: 32 pins
gpio2 at sxipio0: 32 pins
gpio3 at sxipio0: 32 pins
gpio4 at sxipio0: 32 pins
gpio5 at sxipio0: 32 pins
gpio6 at sxipio0: 32 pins
gpio7 at sxipio0: 32 pins
gpio8 at sxipio0: 32 pins
usb2 at ohci0: USB revision 1.0
uhub2 at usb2 configuration 1 interface 0 "Generic OHCI root hub" rev 1.00/1.00 addr 1
usb3 at ohci1: USB revision 1.0
uhub3 at usb3 configuration 1 interface 0 "Generic OHCI root hub" rev 1.00/1.00 addr 1
scsibus1 at sdmmc0: 2 targets, initiator 0
sd1 at scsibus1 targ 1 lun 0: <SD/MMC, CBADS, 0010> removable
sd1: 30007MB, 512 bytes/sector, 61454336 sectors
vscsi0 at root
scsibus2 at vscsi0: 256 targets
softraid0 at root
scsibus3 at softraid0: 256 targets
bootfile: sd0a:/bsd
boot device: sd0
root on sd1a (1593ab2ee369c420.a) swap on sd1b dump on sd1b
```

```bash
# sysctl hw.sensors
hw.sensors.sxits0.temp0=40.20 degC
hw.sensors.axppmic0.temp0=35.00 degC
hw.sensors.axppmic0.volt0=5.13 VDC (ACIN)
hw.sensors.axppmic0.volt1=0.01 VDC (VBUS)
hw.sensors.axppmic0.volt2=4.99 VDC (APS)
hw.sensors.axppmic0.current0=0.11 A (ACIN)
hw.sensors.axppmic0.current1=0.00 A (VBUS)
hw.sensors.axppmic0.indicator0=On (ACIN), OK
hw.sensors.axppmic0.indicator1=Off (VBUS)
```
