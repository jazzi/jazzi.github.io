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

Then create the [Samba configuration](https://www.samba.org/samba/docs/using_samba/ch09.html) */usr/local/etc/smb4.conf* and put some settings there:

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
    valid users = jazzi mike lily
```

As we put */var/log* as *tmpfs* and Samba gonna need to write its logs into */var/log/samba4/* but this directory is not existed and Samba can not create it, so we need to creat it in advance:

`mkdir /var/log/samba4`

As it will disappear after reboot, we need to put this command into */etc/rc.local* to let the system create it automatically.

Ater that we need to enable Samba and start it:

`sysrc samba_server_enable=YES`

`service samba_server start`

So Samba Server is ready and it's time to add some users. By default Samba relies on system users, you need to use command *adduser* to create *system* grade user in advance, then enable it for Samba by:

`pdbedit -a -u jazzi`

After that you can open your MacOS App Finder and connect the server as:

`smb://192.168.0.2`

Also you might want to tune some kernel settings to max files handling. Add the followings suggested by [Davd.io](https://www.davd.io/samba-fileserver-on-freebsd/) into */etc/sysctl.conf*:

You can check another post about [How to access samba without password](/howto-access-samba-without-password.html).

```
kern.maxfiles=25600
kern.maxfilesperproc=16384
net.inet.tcp.sendspace=65536
net.inet.tcp.recvspace=65536
```

## Aria2 settings for FreeBSD

> [aria2](https://aria2.github.io) is a lightweight multi-protocol & multi-source command-line download utility.

Combined with detaching session function of [terminal multiplexers](http://linuxcommand.org/lc3_adv_termmux.php) like *screen* or *tmux*, it can be a very convenient and powerful headless download server.

Its command is not just issuing *aria2* but *aria2c* instead, to install this package:

`pkg install aria2`

There is no default configuration file and you should add the following lines into */home/mike/.aria2/aria2.conf*:

```
#
## aria2 config
#
# man page  = https://aria2.github.io/manual/en/html/
# file path = /home/jazzi/.aria2/aria2.conf
 

# Download Directory: specify the directory all files will be downloaded to.
# When this directive is commented out, aria2 will download the files to the
# current directory where you execute the aria2 binary.
dir=/data/download


# Bit Torrent: If the speed of the incoming data (download) from other peers is
# greater then the peer-speed-limit, then do not allow any more connections
# than max-peers. The idea is to limit the amount of clients our system will
# connect with to reduce our overall load when we are already saturating our
# incoming bandwidth.  Make sure to set the the peer-speed-limit to your
# preferred incoming (download) speed. Speeds must be whole numbers so 5.5M is
# illegal, but 5500K is valid.  For unlimited connections set
# request-peer-speed-limit something high like 10000M (10gig).
 bt-max-peers=0
 bt-request-peer-speed-limit=0


# Bit Torrent: the max upload speed for all torrents combined. Again, only
# whole numbers are valid. We find a global upload limit is more flexible then
# an upload limit per torrent. Zero(0) is unrestricted upload spreeds.
 max-overall-upload-limit=0


# Bit Torrent: When downloading a torrent remove ALL trackers from the listing.
# This is a good way to only use distributed hash table (DHT) and Peer eXchange
# (PeX) for connections. We find start up of the torrent takes a little longer
# with all trackers disabled, but helps reduce the load on trackers.
 bt-exclude-tracker="*"
 bt-external-ip=127.0.0.1


# Bit Torrent: ports and protocols used for bit torrent TCP and UDP
# connections. Make sure DHT is enabled in order to connect to UDP trackers as
# well as negotiating with DHT and PEX peers. Use xboxlive port 3074 for
# obfuscation.
 dht-listen-port=6970
 enable-dht=true
 enable-peer-exchange=true
 listen-port=51413
 bt-enable-lpd=true


# When running aria2 on FreeBSD with ZFS, disable disk-cache due to ZFS's use
# of Adaptive Replacement Cache (ARC). ZFS can also take advantage of the
# "sparse files" format which is significantly faster then pre allocation of
# file space. For other file systems like EXT4 and XFS you can test
# file-allocation with "prealloc" and "falloc" to see which file-allocation
# allows arai2 to start quicker and use less disk I/O.
 disk-cache=0
 file-allocation=none


# Bit Torrent: fully encrypt the negotiation as well and the payload of all bit
# torrent traffic. With this configuration, encryption is required and all old,
# non-encrypted clients are ignored (dropped). This may help avoid some ISPs
# rate limiting P2P clients, but will also reduce the amount of clients aria2
# will talk to.
 bt-force-encryption=true
 bt-min-crypto-level=arc4
 bt-require-crypto=true


# Bit Torrent: Download the torrent file into memory (RAM) if there is no need
# to save the .torrent file itself. This option works with both magnet and
# torrent URL links.
 follow-torrent=mem


# Bit Torrent: The amount of time and the upload-to-download ratio you wish to
# seed to. If either the seed time limit ( minutes ) or the seed ratio is
# reached, torrent seeding will stop. You can set seed-time to zero(0) to
# disable seeding completely.
 seed-ratio=10
 seed-time=60


# Bit Torrent: timeout values for servers and clients.
#bt-tracker-connect-timeout=10
#bt-tracker-interval=900
#bt-tracker-timeout=10


# Bit Torrent: scripts or commands to execute before, during or after a
# download finishes.
# on-bt-download-complete=/path/to/script.sh
# on-download-complete=/path/to/script.pl
# on-download-error=/path/to/script
# on-download-pause=/path/to/script.sh
# on-download-start=/path/to/script.pl
# on-download-stop=/path/to/script


# Network: maximum socket receive buffer in bytes. 1M can sustain 1Gbit/sec.
# Default: 0 which is disabled.
 socket-recv-buffer-size=1M


# Event Multiplexing: set polling to the OS type you are using. For FreeBSD,
# OpenBSD and NetBSD set to "kqueue". For Linux set to "epoll".
 event-poll=kqueue


# Certificate Authority PEM : specify the full path to the OS certificate
# authority pem file to verify the peers. On FreeBSD with OpenSSL the following
# file path is valid. Without a valid pem file aria2 will print out the error,
# "[ERROR] Failed to load trusted CA certificates from no. Cause:
# error:02001002:system library:fopen:No such file or directory"
# The default location for MacOS is /usr/local/openssl/cert.pem
# The default location for OpenBSD is
# The default location for Archlinux is
 ca-certificate=/etc/ssl/cert.pem


# Data Integrity: check the MD5 / SHA256 hash of metalink downloads as well as
# the hash of bit torrent chunks as our client receives them. CPU time is
# reasonably low for the high value of real time verified data. Note:
# check-integrity set as true will show "ERROR - Checksum error detected" for
# magnet links which can be ignored.
#check-integrity=true
 realtime-chunk-checksum=true


# File Names: Resume file downloads if we have a partial copy. Do not rename
# the file or make another copy if the same file is downloaded a second time.
 allow-overwrite=true
 always-resume=true
 auto-file-renaming=false
 continue=true
 remote-time=true


# User Agent: Disable the identification string of our client. If you connect
# to a server which requires a certain id string you can always add one here.
# Trackers should never use client id strings as security authentication or
# access control.
 peer-id-prefix=""
 user-agent=""


# Status Summery messages are disabled since the status of the download is
# updated in real time on the CLI anyways.
 summary-interval=0


# FTP: use passive ftp which is firewall friendly and reuse the ftp data
# connection when asking for multiple resources from the same server for
# efficiency.
 ftp-pasv=true
 ftp-reuse-connection=true


# Metalink: Set the country code to prefer mirrors closest to you. Prefer more
# secure https mirrors over http and ftp servers.
 metalink-language=zh_CN
 metalink-location=cn
 metalink-preferred-protocol=https


# Disconnect from https, http or ftp servers who do not upload data to us
# faster then the specified value. Aria2 will then find another mirror in the
# metalink file which might be quicker. If there are no more mirrors left then
# the current slow mirror is still used. This value is NOT used for bit torrent
# connections though. NOTE: we hope to convince the developer to add a
# lower-speed value or even a minimal client U/D ratio to bit torrent some day
# to kick off leachers too.
 lowest-speed-limit=50K


# Concurrent downloads: Set the number of different servers to download from
# concurrently; i.e. in parallel. If we are downloading a single file then
# split that file into the same amount of streams. Make sure to keep in mind
# that if the amount of parallel downloads times the lowest-speed-limit is
# greater then your total download bandwidth then you will drop servers
# incorrectly. For example, we have ten(10) connections at a minimum of
# 50KiB/sec which equals 500KiB/sec. If our total download bandwidth is not at
# least 500KiB/sec then arai2 will think the mirrors are too slow and drop
# connection slowing down the whole download. Do not set the
# max-connection-per-server greater then three(3) as to avoid abusing a single
# server.
 max-concurrent-downloads=10
 max-connection-per-server=3
 min-split-size=5M
 split=10


# RPC Interface: To access aria2 through XML-RPC API, like using webui-aria2.
#enable-rpc
#rpc-listen-all
#rpc-user=username
#rpc-passwd=passwd


# Daemon Mode: To run aria2 in the background as a daemon. Use daemon mode to
# start aria2 on reboot or when using an RPC interface like webui-aria2.
#daemon=true


#
#
# Reference: the following options arethe developers defaults. We kept them
# here for reference.

# bt-max-open-files=100
# bt-save-metadata=false
# bt-stop-timeout=0
 check-certificate=true
 conditional-get=true
# dht-entry-point="dht.transmissionbt.com:6881"
 dht-file-path=/home/jazzi/.aria2/dht.dat
# dht-message-timeout=10
 disable-ipv6=true
 http-accept-gzip=true
 log=/var/log/aria2.log
 log-level=warn
## BT Tracker server list: https://github.com/ngosang/trackerslist
bt-tracker=udp://tracker.opentrackr.org:1337/announce,udp://open.tracker.cl:1337/announce,udp://tracker.auctor.tv:6969/announce,udp://opentracker.i2p.rocks:6969/announce,https://opentracker.i2p.rocks:443/announce,udp://open.demonii.com:1337/announce,udp://open.stealth.si:80/announce,udp://tracker.torrent.eu.org:451/announce,udp://tracker.moeking.me:6969/announce,udp://exodus.desync.com:6969/announce,udp://tracker.tiny-vps.com:6969/announce,udp://tracker.theoks.net:6969/announce,udp://tracker.skyts.net:6969/announce,udp://retracker01-msk-virt.corbina.net:80/announce,udp://opentracker.io:6969/announce,udp://open.tracker.ink:6969/announce,udp://movies.zsw.ca:6969/announce,udp://explodie.org:6969/announce,udp://bt.ktrackers.com:6666/announce,https://tracker.tamersunion.org:443/announce
### EOF ###
```

### Certificate issue

You might encounter a CA-Certificate fails error as below:

> [SocketCore.cc:933] errorCode=1 SSL/TLS handshake failure: unable to get local issuer certificate

The problem is about the OpenSSL certificate, the FreeBSD installation doesn't have this and you need to install a package as below:

```
$ pkg search ca_root_nss
ca_root_nss-3.93_2             Root certificate bundle from the Mozilla Project
```

`pkg install ca_root_nss`

Or if you just turn off the ca-certificate by the following line, you will still get problem.

> check-certificate=false

### /var/log/aria2.log error

If you use *tmpfs* as */var/log* too, you probably will have such below errors too:

```
$ aria2c /data/download/1.torrent
Exception caught
Exception: [Logger.cc:73] errorCode=1 Failed to open the file /var/log/aria2.log, cause: n/a
```

Above message indicates the file */var/log/aria2.log* is missing, so let's create one

`touch /var/log/aria2.log`

But still the same error shows up, no worries, it's about the privilages, the user needs *write* permission, let's change it by:

`chmod 664 /var/log/aria2.log`

Then woooo, everything goes up, so let's wrap it up and put these two commands into */etc/rc.local*.

## NFSv4 setting

I have problem with MacOS client to connect to FreeBSD NFS server, maybe someday I will dip my toes into it. Anyway here is a link might be useful in the future.

* [macOS X Mount NFS Share / Set an NFS Client](https://www.cyberciti.biz/faq/apple-mac-osx-nfs-mount-command-tutorial/)
* [Share ZFS datasets with FreeBSD NFS](https://wiki.freebsd.org/ZFS/ShareNFS)
* [NFS shares with ZFS](https://klarasystems.com/articles/nfs-shares-with-zfs/)

The whole picture of configuration in file */etc/rc.conf*

```
hostname="rob.yohoyo.home"
#ifconfig_re0="DHCP"
ifconfig_bge1="DHCP"
sshd_enable="YES"
moused_nondefault_enable="NO"
# Set dumpdev to "AUTO" to enable crash dumps, "NO" to disable
dumpdev="AUTO"
zfs_enable="YES"
samba_server_enable="YES"
musicpd_enable="YES"
ympd_enable="YES"
pf_enable="yes"
pflog_enable="yes"
```

According to [manual NFSv4(4)](https://man.freebsd.org/cgi/man.cgi?query=nfsv4&apropos=0&sektion=0&manpath=FreeBSD+14.0-RELEASE+and+Ports&arch=default&format=html), one line need to be existed in */etc/exports*

`V4:	/`

On the MacOS, create */etc/nfs.conf* with followings:

```
# Settings for NFSv4
nfs.client.default_nfs4domain=YOURDOMAIN # needs to be identical with server
nfs.client.mount.options=vers=4,nfc # check man mount_nfs
```

Also NFS share with ZFS enabled on FreeBSD has something special, it doesn't use */etc/exports* but */etc/zfs/exports* instead, this file should not be manually edited but issue the following command:

`zfs set sharenfs="-mapall=jazzi,-network=192.168.31.0/24" pool-name/dataset-name` # option -mallall for write permission

In the MacOS fire the following command to mount it:

`sudo mount -t nfs  192.168.31.249:/data /Users/jazzi/nfs`

After mounted, you can use command `nfsstat -m` to verify it's NFSv4 used or not.

## Set up Owntone with USB DAC

Before we dive in, we need some background about driver or call it module in advance. Normally all loadable modules sit under */boot/kernel*, you can manually load it by command `kldload module_name`, after that you can check the loaded modules by command:

```
# kldstat
Id Refs Address                Size Name
 1   24 0xffffffff80200000  1d345b0 kernel
 2    1 0xffffffff81f35000   5d5958 zfs.ko
 3    1 0xffffffff82e20000     5f00 ig4.ko
 4    1 0xffffffff82e26000     3220 intpm.ko
 5    1 0xffffffff82e2a000     2178 smbus.ko
 6    1 0xffffffff82e2d000     3558 fdescfs.ko
 7    1 0xffffffff82e31000     e5ac snd_uaudio.ko
```
If everything goes ok, you can let the system load it automatically at boot time by add the following line in file */boot/loader.conf*

> snd_uaudio_load="YES"

---

Firstly is to figure out what's hop on USB, in Linux you can simply fire command `lsusb`, in FreeBSD there are three choices at least:

* pciconf -lv
* camcontrol devlist
* usbconfig

```
root@0ob:~ # usbconfig
ugen1.1: <AMD EHCI root HUB> at usbus1, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=SAVE (0mA)
ugen0.1: <AMD XHCI root HUB> at usbus0, cfg=0 md=HOST spd=SUPER (5.0Gbps) pwr=SAVE (0mA)
ugen0.2: <Burr-Brown from TI USB Audio DAC> at usbus0, cfg=0 md=HOST spd=FULL (12Mbps) pwr=ON (20mA)
ugen1.2: <vendor 0x0438 product 0x7900> at usbus1, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=SAVE (100mA)
ugen1.3: <SanDisk Cruzer Blade> at usbus1, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=ON (200mA)
```
Good luck the USB Audio DAC is regonized by the [snd_uaudio driver (4)](https://www.freebsd.org/cgi/man.cgi?query=snd_uaudio), the chip is [TI PCM1794A](https://www.ti.com/product/PCM1794A), to get more details, fire the command:

```
root@rob:~ # dmesg | grep pcm
pcm0: <ATI R6xx (HDMI)> at nid 3 on hdaa0
pcm1: <ATI R6xx (HDMI)> at nid 5 on hdaa0
pcm2: <USB audio> on uaudio0
root@rob:~ # sysctl hw.snd.default_unit=2
hw.snd.default_unit: 0 -> 2
root@rob:~ # mixer -a | grep ^pcm
pcm0:mixer: <ATI R6xx (HDMI)> on hdaa0  (play)
pcm1:mixer: <ATI R6xx (HDMI)> on hdaa0  (play)
pcm2:mixer: <USB audio> at ? kld snd_uaudio (play) (default)
```

So my USB DAC is on the thrid line, in alsa part of */usr/local/etc/owntone.conf* the card="hw:2"
Or
We can set the default playback device by command:

`sysctl hw.snd.default_unit=2` # this number is what we get from above command *cat /dev/sndstat*.


### Check default settings of sound card

So right now how could we figure out which sound card is the default one? As sometimes there is just no sound and don't know what's going on, we can fire the following command to check it out:

```
 $ cat /dev/sndstat 
Installed devices:
pcm0: <ATI R6xx (HDMI)> (play) default
pcm1: <ATI R6xx (HDMI)> (play)
pcm2: <USB audio> (play)
No devices installed from userspace.
```

As above output shows, the default one is pcm0, not the one I want now, so I hit the following command [mixer(8)](https://man.freebsd.org/cgi/man.cgi?query=mixer&sektion=8&format=html) and then run the above command again.


`mixer -d2` # Set USB audio as DAC default

```
 # cat /dev/sndstat 
Installed devices:
pcm0: <ATI R6xx (HDMI)> (play)
pcm1: <ATI R6xx (HDMI)> (play)
pcm2: <USB audio> (play) default
No devices installed from userspace.
```

Now we can try to play some music or just hit command `beep` to get some noise.
According to the [FreeBSD wiki: Sound & Audio](https://wiki.freebsd.org/Sound), with FreeBSD 14.0, the following solution also works:

If everything goes right, we can make this change permanent by adding the next line into */etc/sysctl.conf*

`hw.snd.default_unit=2`

We can check the volume levels with command:

`mixer -f /dev/mixer2`

To set the volume, you can `mixer -f /dev/mixer2 vol 75` # if playback device has been set as defalut, there is no need to specify the device.

You can check the log files either */var/log/owntone.log* or */var/log/messages* if something goes wrong.

### References:

* [USB audio devices on FreeBSD](https://www.davidschlachter.com/misc/freebsd-usb-audio)
* [Audio on FreeBSD - Quick Guide](https://freebsdfoundation.org/resource/audio-on-freebsd-quick-guide/)
* [FreeBSD wiki: Sound & Audio](https://wiki.freebsd.org/Sound)
* [Multimedia of HandBook](https://docs.freebsd.org/en/books/handbook/multimedia/)

**Big Shout, the DAC can get sound alone even motherboard has no sound card built-in.**

### Install Owntone

In order to install **Owntone** you need to install and enable *avahai* and *dbus* in advance.

`pkg install avahi owntone`

Then put the following three lines into */etc/rc.conf* to enable them:

```
dbus_enable="YES"
avahi_daemon_enable="YES"
owntone_enable="YES"
```
After that start them by fire commands:

```
service dbus start
service avahai-daemon start
service owntone start
```

You can check their status by command `service owntone status`.

Hereby is the part of */usr/local/etc/owntone.conf* changed:

```
directories = { "/data/music" }

# Local audio output
audio {
	type = "alsa"
}

```

### Big Problem - no music once DAC turn off

Yes, that's true. Whenever you power off the DAC and turn on again, there is no sound available any more, what's going on? I guess the wrong sound card is used, let's check it out:

```
# sysctl hw.snd | grep default
hw.snd.default_unit: 0
```

As above stated, the default unit is 0, not 2 any more. I turn on the DAC and set it back to 2 again by command `sysctl hw.snd.default_unit=2", then check again:

```
# sysctl hw.snd | grep default
hw.snd.default_unit: 2
```

This is huge problem, every time the DAC is turned off, the default sound card will be back to 0, even it's turned on, it is still 0. What can we do? Never mind, there is a thread on FreeBSD forum [USB DAC as default device?](https://forums.freebsd.org/threads/usb-dac-as-default-device.50489/) offered the solution. Two solutions actually, I chose the following one.

`# sysctl hw.snd.default_auto=2` (see [sound(4)](https://man.freebsd.org/cgi/man.cgi?query=sound&sektion=4&manpath=freebsd-release-ports))

The second solution is by utility [devd(8)](https://man.freebsd.org/cgi/man.cgi?query=devd&sektion=8&apropos=0&manpath=FreeBSD+14.0-RELEASE+and+Ports) which provides a way to have userland prorams run when certain kernel events happen like USB DAC attached. OP Sparklebastard offered his settings like below:

```
# cat /etc/devd/usb-dac.conf

# /etc/devd/usb-dac.conf
attach 100 {
  device-name "pcm2";
  action "sysctl hw.snd.default_unit=2";
};

detach 100 {
  device-name "pcm4";
  action "sysctl hw.snd.default_unit=0";
};
```


The third solution is to shut down the sound card on motherboard in BIOS settings.

---

## Defend with PF firewall

Firstly enable PF & PFlog service

`sysrc pf_enable=YES`
`sysrc pflog_enable=YES`

Then copy pf configuration from /usr/share/example/pf/pf.conf to /etc/pf.conf and edit it according to your needs, what I want is just to restrict SSH access allowed from a specific IP address only and Samba connection from local LAN only. Below is my configuration.

```
cat /etc/pf.conf 
#	$OpenBSD: pf.conf,v 1.34 2007/02/24 19:30:59 millert Exp $
#
# See pf.conf(5) and /usr/share/examples/pf for syntax and examples.
# Remember to set gateway_enable="YES" and/or ipv6_gateway_enable="YES"
# in /etc/rc.conf if packets are to be forwarded between interfaces.

#ext_if="bge0" # Marked 1 on HPE Gen10
int_if="bge1" # Marked 2
#int_if="re0" # PCIe 2.5Gbps
lan_net="192.168.31.0/24"

#table <spamd-white> persist
#table <bruteforce> persist # uncomment it if SSH rule below used.

set skip on lo

#scrub in

#nat-anchor "ftp-proxy/*"
#rdr-anchor "ftp-proxy/*"
#nat on $ext_if inet from !($ext_if) -> ($ext_if:0)
#rdr pass on $int_if proto tcp to port ftp -> 127.0.0.1 port 8021
#no rdr on $ext_if proto tcp from <spamd-white> to any port smtp
#rdr pass on $ext_if proto tcp from any to any port smtp \
#	-> 127.0.0.1 port spamd

#anchor "ftp-proxy/*"
block in
pass out 

# allow ssh connection from specific ip
block return in proto tcp from any to any port 22
#pass in quick on $int_if inet proto tcp from 192.168.31.201 to any port 22 no state
pass in quick on $int_if inet proto tcp from $lan_net to any port 22 no state

# protecting SSH port from brute force attacks which need to decleared the table in the front in advance.
#pass in on $int_if inet proto tcp to port { 1122 } keep state (max-src-conn 15, max-src-conn-rate 3/1, overload <bruteforce> flush global)

# allow samba connection from local network
pass in quick on $int_if inet proto tcp from $lan_net to any port { 139, 445, 8080 }


#pass quick on $int_if no state
#antispoof quick for { lo $int_if }

#pass in on $ext_if proto tcp to ($ext_if) port ssh
#pass in log on $ext_if proto tcp to ($ext_if) port smtp
#pass out log on $ext_if proto tcp from ($ext_if) to port smtp
#pass in on $ext_if inet proto icmp from any to ($ext_if) icmp-type { unreach, redir, timex }
```

---

## ~~Clone USB stick with dd or ddrescue~~

At this point, the whole system is kind of fully ready, to backup I chose to clone another USB stick with same brand same size, so once the USB stick fails, I can remove it and plug another one immediately, 100% no worries left in the dream.

### ~~Clone with command dd~~

Create an image

`dd if=/dev/da0 of=/data/FreeBSD-USB.img bs=4m conv=noerror,sync status=progress` 

Burn the image into new USB stick

`dd if=/data/FreeBSD-USB.img of=/dev/da1 bs=4m`

Then downloaded the image file into my offline USB hard drive, thus you can create a working USB stick anytime you want.

Be sure to double check out the right device number for plugged USB stick by command `camcontrol devlist`.

### ~~Clone USB with command ddrescue~~

[ddrescue](https://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html) is a very powerful and convenient tool for data saving, also good choice for clone USB stick, the command is very straight forward, just type:

`ddrescue /dev/da0 /dev/da1 logfile.map` # da1 is the empty new one

```
 # ddrescue --force /dev/da0 /dev/da1
GNU ddrescue 1.27
Press Ctrl-C to interrupt
     ipos:   61524 MB, non-trimmed:        0 B,  current rate:   2359 kB/s
     opos:   61524 MB, non-scraped:        0 B,  average rate:   4911 kB/s
non-tried:        0 B,  bad-sector:        0 B,    error rate:       0 B/s
  rescued:   61524 MB,   bad areas:        0,        run time:  3h 28m 45s
pct rescued:  100.00%, read errors:        0,  remaining time:         n/a
                              time since last successful read:         n/a
Copying non-tried blocks... Pass 1 (forwards)d
Finished
```

After clone finished, take a snapshot of zroot:

`zfs snapshot -r zroot`

Now we can move on to polish the system with [Jail(8)](https://man.freebsd.org/cgi/man.cgi?query=jail&apropos=0&sektion=0&manpath=FreeBSD+14.0-RELEASE+and+Ports&arch=default&format=html).


## Confirmed both aboved clone methods do not work

After implimented above *dd* or *ddrescue* command, the booting will fail as the partition for booting is not right after dd, we need to partition it in advance with following [gpart(8)](https://man.freebsd.org/cgi/man.cgi?query=gpart&sektion=8&manpath=freebsd-release-ports) command:

`gpart backup usb-disk-1 | gpart restore -F usb-disk-2`

Please be advised, at this moment only the partition table is restored, this action does not affect the content of partitions.

Then as [T-Daemon](https://forums.freebsd.org/threads/install-freebsd-in-external-usb-hdd-disk-with-auto-zfs.85484/) shows up we need to create directories for the efi loader, copy efi loader and set the gpart bootcode.

[Argentum has a working procedure as below](https://forums.freebsd.org/threads/can-zfs-disks-be-cloned-with-dd.68930/post-464902):

```
I have done this few times but not with dd. ZFS has a functions send / receive -

1. connected both drives to computer
2. manually created partition table (including new ZFS and UEFI boot) on new drive using gpart
3. wrote UEFi boot loader on new drive
4. using zfs send and zfs receive transferred old ZFS partition to new drive
5. also had to adjust root mount point on new drive

Trere is another way, I have also tried and happen to like:

1. partitioned and prepared the disk as in previous description
2. added new ZFS partition as a mirror to existing partition using zpool attach
3. allowed ZFS to resilver new disk
4. now I had a working system with mirrored disks
5. disconnected old disk and removed old device from mirror
6. happy :)
```
## Jailed music service mpd with FreeBSD OSS

[Owntone doesn't support OSS](https://github.com/owntone/owntone-server/issues/285), in order to use Alsa you need to install the alsa library with `pkg install alsa-lib`. Actually you need to install a lot of packages if you want go with Owntone, and I want a more slim OS, good news is [mpd](https://www.musicpd.org) is very well supported by FreeBSD OSS - the native default sound system.

By the way, mpd is called **Musicpd** in FreeBSD, the client *mpc* is called **musicpc**.

Firstly a summary of the state of my USB DAC:

```
$ uname -a
FreeBSD 14.0-RELEASE-p5 FreeBSD 14.0-RELEASE-p5 #0: Tue Feb 13 23:37:36 UTC 2024     root@amd64-builder.daemonology.net:/usr/obj/usr/src/amd64.amd64/sys/GENERIC amd64

$ cat /dev/sndstat 
Installed devices:
pcm0: <ATI R6xx (HDMI)> (play)
pcm1: <ATI R6xx (HDMI)> (play)
pcm2: <USB audio> (play) default
No devices installed from userspace.

$ sysctl dev.pcm.2
dev.pcm.2.feedback_rate: 0
dev.pcm.2.mixer.mute_1.desc: 
dev.pcm.2.mixer.mute_1.max: 1
dev.pcm.2.mixer.mute_1.min: 0
dev.pcm.2.mixer.mute_1.val: 0
dev.pcm.2.mixer.vol_0_1.desc: 
dev.pcm.2.mixer.vol_0_1.max: 0
dev.pcm.2.mixer.vol_0_1.min: -16384
dev.pcm.2.mixer.vol_0_1.val: -7209
dev.pcm.2.mixer.vol_0_0.desc: 
dev.pcm.2.mixer.vol_0_0.max: 0
dev.pcm.2.mixer.vol_0_0.min: -16384
dev.pcm.2.mixer.vol_0_0.val: -7209
dev.pcm.2.mode: 3
dev.pcm.2.bitperfect: 0
dev.pcm.2.buffersize: 0
dev.pcm.2.play.vchanformat: s16le:2.0
dev.pcm.2.play.vchanrate: 48000
dev.pcm.2.play.vchanmode: fixed
dev.pcm.2.play.vchans: 1
dev.pcm.2.hwvol_mixer: vol
dev.pcm.2.hwvol_step: 5
dev.pcm.2.%parent: uaudio0
dev.pcm.2.%pnpinfo: 
dev.pcm.2.%location: 
dev.pcm.2.%driver: pcm
dev.pcm.2.%desc: USB audio

$ sysctl hw.usb.uaudio
hw.usb.uaudio.debug: 0
hw.usb.uaudio.buffer_ms: 2
hw.usb.uaudio.default_channels: 0
hw.usb.uaudio.default_bits: 32
hw.usb.uaudio.default_rate: 0
hw.usb.uaudio.handle_hid: 1

$ dmesg | grep uaudio
uaudio0 on uhub1
uaudio0: <Burr-Brown from TI USB Audio DAC, class 0/0, rev 1.10/1.00, addr 1> on usbus0
uaudio0: Play[0]: 48000 Hz, 2 ch, 16-bit S-LE PCM format, 2x2ms buffer.
uaudio0: Play[0]: 44100 Hz, 2 ch, 16-bit S-LE PCM format, 2x2ms buffer.
uaudio0: Play[0]: 32000 Hz, 2 ch, 16-bit S-LE PCM format, 2x2ms buffer.
uaudio0: No recording.
uaudio0: No MIDI sequencer.
pcm2: <USB audio> on uaudio0
uaudio0: HID volume keys found.
```

### Use Jail manager Bastille

Either build the Jail by hand or by tool is ok, [the FreeBSD Documents](https://docs.freebsd.org/en/books/handbook/jails/) has it very well explained even with examples. However I prefer to making life easier, so I chose [Bastille](https://bastillebsd.org) to manage all my Jails. [iocell](https://iocell.readthedocs.io/en/latest/) is another good option too.

Anyway, below is all built with Bastille.

`pkg install bastille`

After installation, configure two settings as below:

```
vim /usr/local/etc/bastille/bastille.conf

## ZFS options
bastille_zfs_enable="YES"                                                ## default: ""
bastille_zfs_zpool="zroot/bastille"                                                 ## default: ""
```

Then create the dataset:

`zfs create zroot/bastille`

Everything is ready, so it's time to bootstrap it:

`bastille bootstrap 14.0-RELEASE update`

At this point, the foundation is finished for coming Jails.

### Build a mpd within FreeBSD Jail

Just build a jail for mpd as below and it will start automatically:

`bastille create mpdjail 14.0-RELEASE 192.168.31.241 bge0`

Then install mpd and ympd:

`bastille pkg mpdjail install musicpd ympd`

Two points need to resolve:

1. How to access music files on host from Jail
2. How to access auido device on host from Jail

Luckily FreeBSD Jails have very smart yet easy way to resolve these problems, and both can be done with Bastille.

The first problem is done with **mount** - a sub command of Bastille:

```
bastille cmd mpdjail mkdir /var/mpd/music  # create directory for mount
bastille mount mpdjail /data/music /var/mpd/music nullfs ro 0 0
[mpdjail]:
Added: /data/music /usr/local/bastille/jails/mpdjail/root//var/mpd/music nullfs ro 0 0

Usage: bastille mount jail_name host_path container_path [filesystem_type options dump pass_number]
```

You can see it makes use of **nullfs**.

The second problem is resolved by making use of [devfs(8)](https://man.freebsd.org/cgi/man.cgi?query=devfs&sektion=8&apropos=0&manpath=FreeBSD+14.0-RELEASE+and+Ports), two steps to do, create the ruleset and apply it.

OK, let's create the ruleset:

```
cat /etc/devfs.rules 
# Devices exposed to jail for music playing
# 
[devfsrules_jail_mpd=6]
add include $devfsrules_hide_all
add path 'mixer*' unhide
add path 'ugen*' unhide  # expose USB DAC
```

Then change *devfs_ruleset=6* as below:

```
# cat /usr/local/bastille/jails/mpdjail/jail.conf 
mpdjail {
  devfs_ruleset = 6;
#  enforce_statfs = 2;
  exec.clean;
  exec.consolelog = /var/log/bastille/mpdjail_console.log;
  exec.start = '/bin/sh /etc/rc';
  exec.stop = '/bin/sh /etc/rc.shutdown';
  host.hostname = mpdjail;
  mount.devfs;
  mount.fstab = /usr/local/bastille/jails/mpdjail/fstab;
  path = /usr/local/bastille/jails/mpdjail/root;
  securelevel = 2;
  osrelease = 14.0-RELEASE;

  interface = bge0;
  ip4.addr = 192.168.31.241;
  
  ip6 = disable;
  allow.mount;
  allow.mount.devfs;
  enforce_statfs = 0;
```

Right now, we can restart the Jail:

`bastille restart mpdjail`

And visit mpd client webUI by any browser URL:

`192.168.31.241:8080`

### Useful links:

1. [Why we're migrating servers from Linux to FreeBSD](https://it-notes.dragas.net/2022/01/24/why-were-migrating-many-of-our-servers-from-linux-to-freebsd/)
2. [Managing Jails in FreeBSD with Bastille](https://alfaexploit.com/en/posts/managing_jails_in_freebsd_with_bastille/#mounting-directories)
3. [Bastille Tips](https://adriel-tech.github.io/bastillebsd/freebsd13/2021/11/12/BastilleBSD-Tips.html)
4. [Easy and lightweight jails with BastilleBSD](https://hackacad.net/freebsd/2021/01/18/easy-freebsd-jail-management-bastille.html)
5. [Jails - Accessing devices from Bastille](https://forums.freebsd.org/threads/jails-accessing-devices-from-bastille.79781/)
6. [FreeBSD Audio](https://meka.rs/blog/2021/10/12/freebsd-audio/)

## Manully set up Lyrion Music Server in Jail

Still encountering some minor problems with *Bastille*, so why not abandon Jail Manager Tool and work it out manully? Also what I need is not tons of Jails, it's so simple, just two or three Jails only.

[Chapter 17. Jails and Containers](https://docs.freebsd.org/en/books/handbook/jails/) in the FreeBSD Handbook is an excellent instruction, just follow it and create a Jail named **lms**.

To simplify the coming Jails, here is my /etc/jail.conf

```
$ cat /etc/jail.conf

# Global settings applied to all jails

interface = "bge0";
host.hostname = "${name}";
path = "/usr/local/jails/containers/${name}";
ip4 = inherit;
mount.fstab = "/usr/local/jails/containers/${name}.fstab";

exec.start = "/bin/sh /etc/rc";
exec.stop = "/bin/sh /etc/rc.shutdown";
exec.consolelog = "/var/log/jail_console_${name}.log";
allow.raw_sockets;
exec.clean;
mount.devfs;


# The jail definition for cups
cups {
#  mount.fstab = "";
  # PASSTHROUGHT PRINTER
  devfs_ruleset = 10;
  osrelease = 14.0-RELEASE;
}

# The jail definiton for lms - Lyrion Music Server
lms {
  osrelease = 14.0-RELEASE;
#  mount.fstab = "";
}
```

The only problem needs to work out is how to let the Jailed service access host's music files, the solution is to make use of [nullfs](https://man.freebsd.org/cgi/man.cgi?query=nullfs&apropos=0&sektion=0&manpath=FreeBSD+14.0-RELEASE+and+Ports&arch=default&format=html) and create a special *fstab* file as below.

```
$ cat /usr/local/jails/containers/lms.fstab 
/data/music /usr/local/jails/containers/lms/var/music nullfs rw,late 0 0
/var/cache/pkg /usr/local/jails/containers/lms/var/cache/pkg nullfs rw,late 0 0 # do it as your wish
```

Sure you need to get into the jail to create the mount point */var/music* in advance. Then restart jail with following command and you are able to read these music files in the Jail.

`service jail restart lms`

## Common Issues

### pkg install failed: size mismatch

After OS minor upgrade, the command `pkg install` got problem as below:

`pkg: cached package <portname>: size mismatch, cannot continue`

The solution is [here](https://github.com/freebsd/pkg/issues/902) to remove the repo* from /var/db/pkg as below:

`rm -rf /var/db/pkg/repo*` then `pkg update`
