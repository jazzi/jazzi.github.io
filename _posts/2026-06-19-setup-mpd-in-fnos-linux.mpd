---
layout: post
---

[fnos](https://www.fnos.com) is based on Debian Linux, so no big deal to set up this [mpd](https://www.musicpd.org) music player daemon, just `sudo apt install mpd` to install and do the following configurations.

## Important configurations

Two configuration files:

* /etc/mpd.conf
* ~/.config/mpd/mpd.conf

I chose the first one as it is system wide, good for NAS haha. Some items need to look up:

1. music_directory		"/var/lib/mpd/music"
2. playlist_directory		"/var/lib/mpd/playlists"
3. db_file			"/var/lib/mpd/tag_cache"
4. #log_file			"/var/log/mpd/mpd.log"
5. state_file			"/var/lib/mpd/state"
6. sticker_file                   "/var/lib/mpd/sticker.sql"
7. user				"mpd"
8. #group				"nogroup"
9. bind_to_address			"localhost" ## For network
10. #bind_to_address		"/run/mpd/socket" ## And for Unix Socket
11. #port				"6600"

And an example of an ALSA output:
#
#audio_output {
#	type		"alsa"
#	name		"My ALSA Device"
##	device		"hw:0,0"	# optional
##	mixer_type      "hardware"	# optional
##	mixer_device	"default"	# optional
##	mixer_control	"PCM"		# optional
##	mixer_index	"0"		# optional
#}

---

## Set up music directory

```
$ ls -l /var/lib/mpd
total 8
drwxr-xr-x 2 root root  4096 Apr  9  2023 music
drwxr-xr-x 2 mpd  audio 4096 Apr  9  2023 playlists
```

As the music files are already stored in the NAS, so I wanna create a soft link instead of moving all music files into */var/lib/mpd/music*.

```
$ sudo rmdir /var/lib/mpd/music # remove it
$ sudo ln -s /vol2/1000/music /var/lib/mpd/music # create soft link

$ sudo rmdir /var/lib/mpd/playlists/
$ sudo ln -s /vol2/1000/music/playlist /var/lib/mpd/playlists

Then check it again
$ ls -l /var/lib/mpd
total 4
lrwxrwxrwx 1 root root    16 Jun 20 21:47 music -> /vol2/1000/music
drwxr-xr-x 2 mpd  audio 4096 Apr  9  2023 playlists
```

## Configure alsa for USB DAC

I chose *ALSA* for output thus need to install one package:

`sudo apt install alsa-utils`

Then commands below would be available for figuring out USB DAC

```
$ sudo lsusb
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 003: ID 13d3:3621 IMC Networks Wireless_Device
Bus 003 Device 017: ID 30be:0106 Schiit Audio I'm Fulla Schiit
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

$ sudo aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: Schiit [I'm Fulla Schiit], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: PCH [HDA Intel PCH], device 0: ALC897 Analog [ALC897 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: PCH [HDA Intel PCH], device 3: HDMI 0 [HDMI 0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: PCH [HDA Intel PCH], device 7: HDMI 1 [HDMI 1]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: PCH [HDA Intel PCH], device 8: HDMI 2 [HDMI 2]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: PCH [HDA Intel PCH], device 9: HDMI 3 [HDMI 3]
  Subdevices: 1/1
  Subdevice #0: subdevice #0

$ sudo aplay --list-pcm
null
    Discard all samples (playback) or generate zero samples (capture)
default
    Default Audio Device
sysdefault
    Default Audio Device
hw:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    Direct hardware device without any conversions
plughw:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    Hardware device with all software conversions
default:CARD=Schiit
    I'm Fulla Schiit, USB Audio
    Default Audio Device
sysdefault:CARD=Schiit
    I'm Fulla Schiit, USB Audio
    Default Audio Device
front:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    Front output / input
surround21:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    2.1 Surround output to Front and Subwoofer speakers
surround40:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    4.0 Surround output to Front and Rear speakers
surround41:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    4.1 Surround output to Front, Rear and Subwoofer speakers
surround50:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    5.0 Surround output to Front, Center and Rear speakers
surround51:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    5.1 Surround output to Front, Center, Rear and Subwoofer speakers
surround71:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    7.1 Surround output to Front, Center, Side, Rear and Woofer speakers
iec958:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    IEC958 (S/PDIF) Digital Audio Output
dmix:CARD=Schiit,DEV=0
    I'm Fulla Schiit, USB Audio
    Direct sample mixing device
hw:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    Direct hardware device without any conversions
hw:CARD=PCH,DEV=3
    HDA Intel PCH, HDMI 0
    Direct hardware device without any conversions
hw:CARD=PCH,DEV=7
    HDA Intel PCH, HDMI 1
    Direct hardware device without any conversions
hw:CARD=PCH,DEV=8
    HDA Intel PCH, HDMI 2
    Direct hardware device without any conversions
hw:CARD=PCH,DEV=9
    HDA Intel PCH, HDMI 3
    Direct hardware device without any conversions
plughw:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    Hardware device with all software conversions
plughw:CARD=PCH,DEV=3
    HDA Intel PCH, HDMI 0
    Hardware device with all software conversions
plughw:CARD=PCH,DEV=7
    HDA Intel PCH, HDMI 1
    Hardware device with all software conversions
plughw:CARD=PCH,DEV=8
    HDA Intel PCH, HDMI 2
    Hardware device with all software conversions
plughw:CARD=PCH,DEV=9
    HDA Intel PCH, HDMI 3
    Hardware device with all software conversions
default:CARD=PCH
    HDA Intel PCH, ALC897 Analog
    Default Audio Device
sysdefault:CARD=PCH
    HDA Intel PCH, ALC897 Analog
    Default Audio Device
front:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    Front output / input
surround21:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    2.1 Surround output to Front and Subwoofer speakers
surround40:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    4.0 Surround output to Front and Rear speakers
surround41:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    4.1 Surround output to Front, Rear and Subwoofer speakers
surround50:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    5.0 Surround output to Front, Center and Rear speakers
surround51:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    5.1 Surround output to Front, Center, Rear and Subwoofer speakers
surround71:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    7.1 Surround output to Front, Center, Side, Rear and Woofer speakers
hdmi:CARD=PCH,DEV=0
    HDA Intel PCH, HDMI 0
    HDMI Audio Output
hdmi:CARD=PCH,DEV=1
    HDA Intel PCH, HDMI 1
    HDMI Audio Output
hdmi:CARD=PCH,DEV=2
    HDA Intel PCH, HDMI 2
    HDMI Audio Output
hdmi:CARD=PCH,DEV=3
    HDA Intel PCH, HDMI 3
    HDMI Audio Output
dmix:CARD=PCH,DEV=0
    HDA Intel PCH, ALC897 Analog
    Direct sample mixing device
dmix:CARD=PCH,DEV=3
    HDA Intel PCH, HDMI 0
    Direct sample mixing device
dmix:CARD=PCH,DEV=7
    HDA Intel PCH, HDMI 1
    Direct sample mixing device
dmix:CARD=PCH,DEV=8
    HDA Intel PCH, HDMI 2
    Direct sample mixing device
dmix:CARD=PCH,DEV=9
    HDA Intel PCH, HDMI 3
    Direct sample mixing device
```

So as you can see from above output, my USB DAC is **hw:CARD=Schiit,DEV=0**, thus to set ALSA output configuration as below:

```
audio_output {
       type            "alsa"
       name            "I'm Fulla Schiit"
       device          "hw:0,0"        # optional
      mixer_type      "hardware"      # optional
      mixer_device    "default"       # optional
      mixer_control   "PCM"           # optional
      mixer_index     "0"             # optional
}
```
For *mixer_control*, check output of command `sudo alsamixer` while it confirms the answer if "PCM".

## Create missing files

```
$ sudo touch /var/lib/mpd/tag_cache
$ sudo touch /var/lib/mpd/state
$ sudo chown mpd:audio /var/lib/mpd/state
$ sudo chown mpd:audio /var/lib/mpd/tag_cache
$ ls -l /var/lib/mpd
total 96
lrwxrwxrwx 1 root root     16 Jun 20 21:47 music -> /vol2/1000/music
drwxr-xr-x 2 mpd  audio  4096 Apr  9  2023 playlists
-rw-r--r-- 1 mpd  audio   191 Jun 21 20:44 state
-rw-r--r-- 1 mpd  audio 12288 Jun 21 20:40 sticker.sql
-rw-r--r-- 1 mpd  audio 77160 Jun 21 20:42 tag_cache
```

Finally it's time to start MPD Service as below:

```
$ sudo systemctl enable mpd.service
$ sudo systemctl start mpd.service
$ sudo systemctl status mpd.service
● mpd.service - Music Player Daemon
     Loaded: loaded (/lib/systemd/system/mpd.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-06-21 20:44:09 CST; 6s ago
TriggeredBy: ○ mpd.socket
       Docs: man:mpd(1)
             man:mpd.conf(5)
             file:///usr/share/doc/mpd/html/user.html
   Main PID: 354482 (mpd)
      Tasks: 3 (limit: 9124)
     Memory: 19.1M
        CPU: 138ms
     CGroup: /system.slice/mpd.service
             └─354482 /usr/bin/mpd --systemd
```

Till now everything seems fine, let's do a final check with the client, see if you can get the music out.

```
sudo apt install mpc # install command line client

$ mpc update
Updating DB (#1) ...
volume:  0%   repeat: off   random: off   single: off   consume: off

$ mpc stats
Artists:    425
Albums:     174
Songs:     2099

Play Time:    0 days, 0:00:00
Uptime:       0 days, 0:07:41
DB Updated:   Sun Jun 21 20:50:30 2026
DB Play Time: 28 days, 14:47:20

$ mpc listall # list all music files in the database
$ mpc add "王菲合集/王菲《天空》 K2HD日本首批限量版[低速原抓WAV+分轨]/01  天空.wav" # add one music into the queue
$ mpc play
$ mpc status
王菲合集/王菲《天空》 K2HD日本首批限量版[低速原抓WAV+分轨]/01  天空.wav
[playing] #1/1   0:19/4:24 (7%)
volume:  0%   repeat: off   random: off   single: off   consume: off
```

