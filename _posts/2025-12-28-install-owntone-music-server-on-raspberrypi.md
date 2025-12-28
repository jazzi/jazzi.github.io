---
layout: post
---

For this Raspberry Pi 3B, to easy myself, finally installed Raspberry Pi OS.

## Owntone music server

Package owntone is not availbe in the official repository, so got to add a special repository.

1. wget -q -O - https://raw.githubusercontent.com/owntone/owntone-apt/refs/heads/master/repo/rpi/owntone.gpg | sudo gpg --dearmor --output /usr/share/keyrings/owntone-archive-keyring.gpg  ## Add repository key
2. vi /etc/apt/sources.list.d/owntone.list ## Add followings into it

```
# OwnTone APT archive for Raspberry Pi OS
# See https://www.raspberrypi.org/forums/viewtopic.php?t=49928
deb [signed-by=/usr/share/keyrings/owntone-archive-keyring.gpg] http://www.gyfgafguf.dk/raspbian/owntone/ trixie contrib
#deb-src [signed-by=/usr/share/keyrings/owntone-archive-keyring.gpg] http://www.gyfgafguf.dk/raspbian/owntone/ trixie contrib
```

Then `sudo apt update & sudo apt install owntone`

