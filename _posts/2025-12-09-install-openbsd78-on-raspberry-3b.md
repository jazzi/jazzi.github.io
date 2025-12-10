---
layout: post
---

My first thought was to install FreeBSD on this Raspberry Pi 3B, unfortunatelly FreeBSD doesn't support the built-in WiFi, so have moved to OpenBSD 7.8.

This installation instruction mostly comes from [mipam007](https://github.com/mipam007/howto/blob/main/openbsd-on-raspberrypi3bplus.md).

Host computer is MacOS.

## Update RPI3b firmware

Download Raspberry Pi OS lite and write to SD card, the tool for burning image is Raspberry Imager which can enalbe SSH, set up password even WiFi, recommended.

1. sudo echo "enable_uart=1" >> /Volumes/bootfs/config.txt
2. sudo touch /Volumes/bootfs/ssh # create this empty file or SSH won't work
3. Start RPI & ssh to it
4. sudo apt update -y
5. sudo apt full-upgrade -y
6. sudo reboot
7. sudo apt autoremove
8. sudo rpi-update # Update rpi firmware

*Regarding this rpi-update, it requires to download from GitHub and there is problem with the conneciton here, so I chose to **Download it in advance** and update it locally as below:

```
curl -L https://codeload.github.com/raspberrypi/rpi-firmware/zip/refs/heads/master -o rpi-firmware-master.zip

#1. 切换到root用户（第一次切到root记得用sudo passwd root**）
su

# 2. 创建一个.rpi-firmware目录，然后进入并解压
mkdir /root/.rpi-firmware
cd /root/.rpi-firmware && unzip /root/rpi-firmware-master.zip

cp -r ./rpi-firmware-master/* ./

# 执行本地更新
UPDATE_SELF=0 SKIP_DOWNLOAD=1 rpi-update

# 重启
reboot

```

## WiFi blocked

Login again and you will see the following error:

> Wi-Fi is currently blocked by rfkill.
>
> Use raspi-config to set the country before use.

**rfkill** = radio frequency kill, is a manage system for WiFi and BulueTooth. Hereby explain what is [rfkill](https://wireless.docs.kernel.org/en/latest/en/users/documentation/rfkill.html):

> rfkill is a small userspace tool to query the state of the rfkill switches, buttons and subsystem interfaces. Some devices come with a hard switch that lets you kill different types of RF radios: 802.11 / Bluetooth / NFC / UWB / WAN / WIMAX / FM. 

So what need to do is to set the country code through command *raspi-config* and choose **Change wlan country**.
