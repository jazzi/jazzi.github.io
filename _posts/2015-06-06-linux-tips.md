---
layout: post
---
This is all about Linux tips that will be constantly updated.

## Howto let Virtualbox boot from ISO ##
Create a new machine, then under *Settings > Storage*, there are 2 controllers (IDE and SATA), pick IDE one and click the Drive Icon there and could *"Add a CD/DVD device"*, then navigate to the iso file and save those settings.

Set the boot option to begin with CD/DVD, so your machine can take off.


## Howto get USB back after dd ##
You can type `dd if=/iso_file of=/dev/sdx bs=4M && sync` to burn an image into USB stick, hereby howto reverse it:

1. Zero its first 512 bytes:
	# dd count=1 bs=512 if=/dev/zero of=/dev/sdx && sync

2. Then install dosfstools and FAT32 and run:
	# cfdisk /dev/sdx
	# mkfs.vfat -F32 /dev/sdx1
	# dosfslabel /dev/sdx1	The_Label_U_Like


