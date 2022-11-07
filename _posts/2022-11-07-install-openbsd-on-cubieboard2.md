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

I bought a serial cable and solution 2 didn't work, don't bad line or what, anyway I plug on keyboard and monitor, it brings out the text on screen, but there is a trick, the console won't work in that way only, you need to set tty to framebuffer as below.

```text
>> OpenBSD/armv7 BOOTARM 1.14
boot> set tty fb0
switching console to fb0
```

*Interesting part is OpenBSD 7.1 didn't work, luckily version 7.2 released soon.*

*Another notice is installing software sets through USB stick caused a problem, after installation finished, the reboot didn't work. So I used **http** instead of **local**, that's giving up USB stick.*

The rest is nothing special, just follow the [instructions on OpenBSD](https://ftp.openbsd.org/pub/OpenBSD/7.2/armv7/INSTALL.armv7).

```text
Location of sets = http
HTTP proxy URL = mirrors.bfsu.edu.cn
HTTP Server = mirrors.bfsu.edu.cn
Server directory = OpenBSD/7.2/armv7
What timezone are you in = Asia/Shanghai
```

## Format new hard drive

My drive was formatted as filesystem ext4 and used under Armbian Linux, now moved to OpenBSD, both partition label and filesystem do not work any more, so need to destroy and create new MBR and FFS as follows:

```text
#fdisk -i sd0
#disklabel -E sd0
#newfs sd0i (given new partition lable is "i")
```

## Set up NFS service

The trick is, /etc/exports file has different options between Linux and OpenBSD, you can not just copy these Linux options to here, for example, option -rw is not permitted on OpenBSD as it is the default setting, so be careful to read the manual page of [/etc/exports](https://man.openbsd.org/exports).

Here is my exports file:

```text
/mnt/hdd	-alldirs -network=192.168.0 -mask=255.255.255.0
```

Just follow [Using NFS on FAQ](https://www.openbsd.org/faq/faq6.html#NFS) and everything should be fine.

Above is server part, below is client part.

* MacOS - it needs a special option for mount, that's **resvport**, I put a line into my */Users/robin/.zshrc* file. 

```text
# Alias to mount NFS server on Cubieboard2
alias nfs="sudo mount -t nfs -o resvport,wsize=32768,rsize=32768 192.168.0.231:/mnt/hdd /Users/robin/nfs"
alias unfs="sudo umount /Users/robin/nfs"
```

* Linux - nothing special, just put the mount into /etc/fstab

The NFS performance is somewhat connected to option *wsize* and *rsize*, 32768/1024=32k, you can tunning it by following this [Optimizing NFS Performance guide](https://nfs.sourceforge.net/nfs-howto/ar01s05.html) and find your best number.

## Troubleshooting

1. I/O Errors - add option *nolock*
2. mount_nfs: can't mount with remote locks when server is not running rpc.statd: RPC prog. not avail - add the following into */etc/rc.local*

```text
#!/bin/csh
#
# MacOS needs rpc.statd
rpc.statd
```

## Reference resource

* [Preparing microSD on MacOS](https://www.tumfatig.net/2018/running-openbsd-on-raspberry-pi-3/)
* [OpenBSD/armv7 on the CubieBoard2](https://www.cambus.net/openbsd-armv7-on-the-cubieboard2/)
* [Installing OpenBSD on Raspberry Pi 3](https://dev.to/spacial/installing-openbsd-7-on-raspberry-pi-3-1f98)
