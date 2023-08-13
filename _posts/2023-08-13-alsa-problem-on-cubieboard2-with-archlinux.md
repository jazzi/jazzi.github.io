---
layout: post
---

Arch Linux is so light that I think might be the best option for old Cubieboard2 single board, so I followed the [Archlinux installation instruction](https://archlinuxarm.org/platforms/armv7/allwinner/cubieboard-2) and get it working.

## No Sound - ALSA problem

After `ssh alarm@192.168.1.5` and enter password "alarm", then change to root by `su -` with password "root", I found everything is fine except the sound.

So add user alarm to group "audio":

`usermod -a -G audio alarm`

Then type `exit` to back to user "alarm"

```
[jazzi@alarm ~]$ aplay -L
null
    Discard all samples (playback) or generate zero samples (capture)
default:CARD=sun4icodec
    sun4i-codec, CDC PCM Codec-0
    Default Audio Device
sysdefault:CARD=sun4icodec
    sun4i-codec, CDC PCM Codec-0
    Default Audio Device
```

Above output means the audio card driver is loaded and found by ALSA.

Then install a tool package:

`pacman -S alsa-utils`

After that you can set the sound:

```
[jazzi@alarm ~]$ alsamixer
Card: sun4i-codec
Use left and right arrow to choose the item and unmute all items by stroke M
```

**Remember to unmute all items**

I ever encountered below errors:

```
[jazzi@alarm ~]$ aplay /usr/share/sound/alsa/Front_Center.wav 
Playing WAVE 'Front_Center.wav' : Signed 16 bit Little Endian, Rate 44100 Hz, Stereo
aplay: pcm_write:2146: write error: Input/output error
```

Finally the problem is one item named "Right Mixer Right DAC" is muted, unmute it solved the problem.

---

## Configure the system

1. Add user alarm to *sudoer* by `visudo` after package "sudo" installed
2. Add SSH Key to the server by run command in client side `ssh-copy-id alarm@192.168.1.5` and copy this key to root by `sudo cp .ssh/authorized_keys /root/.ssh`
3. Now you can login as root to change user name `usermode -l jazzi -d /home/jazzi -m alarm` and group name `groupmod -n jazzi alarm`, actually it's all about /etc/passwd, /etc/group, /etc/shadow and /home/alarm
4. Set the mirror by edit /etc/pacman.d/mirrorlist and add server *Server = https://mirrors.bfsu.edu.cn/archlinuxarm/$arch/$repo*
5. Edit file /etc/locale.gen and uncomment en_US-UTF8 then run `locale-gen` and `localectl set-locale en_US.UTF8`
6. Set timezone by run `timedatectl set-timezone Asia/Shanghai`
7. Add user jazzi to group **audio** by `usermod -a -G audio jazzi`
8. Add user jazzi to *sudoer* by `visudo` after package "sudo" installed
