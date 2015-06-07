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

    `# dd count=1 bs=512 if=/dev/zero of=/dev/sdx && sync`

2. Then install dosfstools and FAT32 and run:

    `# cfdisk /dev/sdx`

    `# mkfs.vfat -F32 /dev/sdx1`

    `# dosfslabel /dev/sdx1 The_Label_U_Like`

## Howto ssh ubuntu in VirtualBox ##
If you have a Linux machine running in Virtualbox, no matter your host is Linux or not, you can login to the guest by ssh without password.

1. Shutdown the Virtualbox guest
2. On the Virtualbox go to *File -> Preferences -> Network -> Host-only Networks* add one
3. Configure the guest, *Settings -> Network -> Adapter 2*, *Enable Network Adapter*, pick *Host-only Adapter* for *Attached to*
4. Edit the /etc/network/interfaces file and add following block:

>autho eth1

>iface eth1 inet static

>address 192.168.56.110

>netmask 255.255.255.0

5. Then restart the networking using command `/etc/init.d/networking`
6. Install openssh-server `apt-get install openssh-server`
7. Now restart the Virtualbox	 guest

8. Edit host's /etc/hosts, `sudo vim /etc/hosts` and add a line *192.168.56.110  guest*

#### The next stage is about ssh login with no need of password by using RSA key ####
1. On the host machine, generate RSA key by `ssh-keygen -t rsa`
2. Copy that public key onto guest machine per `scp ~/.ssh/id_rsa.pub You_User_Name@guest:`  ## Don't forget the ":" in the end
3. Then login `ssh Your_User_Name@guest`
4. `cat id_rsa.pub >> ~/.ssh/authorized.keys`
5. Exit the ssh session and take a try of the new world!!

