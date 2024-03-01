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

## Install FreeBSD on USB stick

There is a 128G SSD there, it's good choice for OS installation however I decided to install the system on a USB stick based on the following reasons:

1. There is an internal USB 2.0 port and USB 2.0 flash memory stick is cheap
2. Easier to clone a USB stick as cold offline backup, once one failes I can just plug another one
3. It's a home NAS server, not much services and workloads on it
4. The above 128G SSD could be a **cache** drive to improve reading speed a lot
5. There is perfect solution to decrease USB **write** 

### How to decrease USB write?

1. Replace default /zroot/tmp as **tmpfs**
  * Delete */zroot/tmp*: `zfs destroy /zroot/tmp`
  * Add a line into */etc/fstab*: `echo "tmpfs	/tmp	tmpfs	rw,mode=1777,size=128m	0	0" >> /etc/fstab`
2. Change **PKG** cache location
  * Put `mkdir -p /tmp/cache/pkg` in */etc/rc.local*
  * Configure */usr/local/etc/pkg.conf* and add the following three lines
      * PKG_CACHEDIR = "/tmp/cache/pkg";
      * AUTOCLEAN = true;
      * REPO_AUTOUPDATE = false;
3. Disable some services and daemons in */etc/rc.conf*
  * sendmail_enable = "NONE"
  * hostid_enable = "NO"
  * ~~syslogd_enable = "NO"~~ # Be careful with this as all logs are disabled
  * ~~newsyslod_enable = "NO"~~
4. Disable some routine periodic services in "/etc/periodic.conf"
  * weekly_locate_enable = "NO"
  * weekly_whatis_enable = "NO" 
  * daily_status_enable = "NO"
  * daily_status_security_enable = "NO"
5. Disable SWAP

As this machine now has 8G memory even can be max 32G later, besides its role is just small home NAS to serve file sharing and music server or maybe video streaming server, should not a memory consuming monster, so SWAP is disabled by setting to 0.

Once the FreeBSD Installer steps into *Allocating Disk Space*, four options shown and I chose the first one:

* Auto (ZFS)	Guided Root-on-ZFS
* Auto (UFS)	Guided UFS Disk Setup
* Manual	Manual Disk Setup (experts)
* Shell		Open a shell and partition by hand

The **Guided Partitioning Using Root-on-ZFS** has the following configureation options:

* \>>> Install		Proceed with Installation
* T Pool Type/Disks:	Stripe: 0 disks
* \- Rescan Devices	*
* \- Disk Info		*
* N Pool Name		zroot
* 4 Force 4k Sectors?	YES
* E Encrypt Disks?	NO
* P Partition Scheme	GPT (BIOS+UEFI)
* S Swap Size		2g
* M Mirror Swap?	NO
* W Encruypt Swap?	NO

I just change the *2g* into zero 0 to disable swap and pick the USB 2.0 stick for **Pool Type/Disks**.

### Update on tmpfs and syslogd service

Disable *syslogd* causes problem for user to login though no problem with remote SSH Login, I decided to keep service *syslogd* and *newsyslog* running and change */var/log* into *tmpfs* by adding the following line into */etc/fstab*

```
tmpfs  /var/log  tmpfs  rw,mode=0755,size=128m  0  0
```

And delete */zroot/var/log* by issuing command `zfs destroy zroot/var/log` but you may encounter file system busy error problem as daemon *syslogd* is still running and writing into this file system, the trick is forcely umount it by command:

`umount --force /zroot/var/log`

Or disable *syslogd* in */etc/rc.conf*:

```
syslogd_enable = "NO"
newsyslog = "NO"
```

Then a reboot will disable both services and you are ok to destroy */zroot/var/log*.

At this point, most of your log files will be in the memory file tmpfs, it's better to sync to somewhere of the disk monthly or weekly, you can put the following command into *cron*:

`rsync -avx --delete /var/log/ /somewhere`

## SSH takes too long time to connect

This problem bothered me, everytime I try to connect the FreeBSD, it takes almost 20 seconds to get connected and I know it should be less than 1s. Adding the following line into */etc/ssh/sshd_config* resolved this problem.

> UseDNS no

## Set up data pool

Firstly is to identiy which disk is attached to which port and which port refers to which disk label.

`camcontrol devlist`
`glabel status`

It shows all attached labels and tells /dev/ada0 is the data drive, so set the data pool up.

`zpool create data /dev/ada0`

This sets up one drive *stripe* type pool, it can be transferred to *mirror* type pool later if one more drive added.

Then I added the SSD cache drive as [L2ARC](http://www.brendangregg.com/blog/2008-07-22/zfs-l2arc.html) which is /dev/ada1

`zpool add data cache /dev/ada1`

Then command `zpool status` indicated everthing is ok.

After that I created several *dataset* as follows:

`zfs create data/tv`
`zfs create data/music`

At this point, the base system is ready, let's install some services.

## Install Samba file sharing server

`pkg search samba`
`pkg install samba416`

Then create the Samba configuration */usr/local/etc/smb4.conf* and put some settings there:

```
[global]
    workgroup = WORKGROUP 
    realm = yo.home
    netbios name = NAS

[data]
    path = /data
    public = no
    writable = yes
    printable = no
    guest ok = no
    valid users = jazzi
```

As we put */var/log* as *tmpfs* and Samba gonna need to write its logs into */var/log/samba4/* but this directory is not existed and Samba can not create it, so we need to creat it in advance:

`mkdir /var/log/samba4`

As it will disappear after reboot, we need to put this command into */etc/rc.local* to let the system create it automatically.

Ater that we need to enable Samba and start it:

`sysrc samba_server_enable=YES`
`service samba_server start`

So Samba Server is ready and it's time to add some users. By default Samba relies on system users, you need to use command *adduser* to create *system* grade user in advance, then enable it for Samba by:

`pdedit -a -u jazzi`

After that you can open your MacOS App Finder and connect the server as:

`smb://192.168.0.2`

Also you might want to tune some kernel settings to max files handling. Add the followings into */etc/sysctl.conf*:

```
kern.maxfiles=25600
kern.maxfilesperproc=16384
net.inet.tcp.sendspace=65536
net.inet.tcp.recvspace=65536
```
