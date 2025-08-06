---
layout: post
---

I have an Android powered TV which support Samba but it doesn't support user/password authentication which means there is no place to input username and password.

>The most common settings are security = share and security = user. If the clients use usernames that are the same as their usernames on the FreeBSD machine, user level security should be used. This is the default security policy and it requires clients to first log on before they can access shared resources.
>
>In share level security, clients do not need to log onto the server with a valid username and password before attempting to connect to a shared resource. This was the default security model for older versions of Samba.

Above words comes from the FreeBSD Handbook, it just indicates *secureity=share * is deprecated, I tried and it doesn't work. So I go back to the [Samba Official Manual](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html) and found the following paremeter:

```
map to guest (G)

This parameter can take four different values, which tell smbd(8) what to do with user login requests that don't match a valid UNIX user in some way.

The four settings are :

Never - Means user login requests with an invalid password are rejected. This is the default.

Bad User - Means user logins with an invalid password are rejected, unless the username does not exist, in which case it is treated as a guest login and mapped into the guest account.

Bad Password - Means user logins with an invalid password are treated as a guest login and mapped into the guest account. Note that this can cause problems as it means that any user incorrectly typing their password will be silently logged on as "guest" - and will not know the reason they cannot access files they think they should - there will have been no message given to them that they got their password wrong. Helpdesk services will hate you if you set the map to guest parameter this way :-).

Bad Uid - Is only applicable when Samba is configured in some type of domain mode security (security = {domain|ads}) and means that user logins which are successfully authenticated but which have no valid Unix user account (and smbd is unable to create one) should be mapped to the defined guest account. This was the default behavior of Samba 2.x releases. Note that if a member server is running winbindd, this option should never be required because the nss_winbind library will export the Windows domain users and groups to the underlying OS via the Name Service Switch interface.

Note that this parameter is needed to set up "Guest" share services. This is because in these modes the name of the resource being requested is not sent to the server until after the server has successfully authenticated the client so the server cannot make authentication decisions at the correct time (connection to the share) for "Guest" shares.

Default: map to guest = Never

Example: map to guest = Bad User
```

I set *map to guest = bad user* and restart it by command `service samba_server restart`, hey it works.

Hereby is my full */usr/local/etc/smb4.conf*, this [link](https://www.server-world.info/en/note?os=FreeBSD_14&p=samba&f=1) is quite helpful too.

---

And the full configurations:

```
cat /usr/local/etc/smb4.conf
[global]
    unix charset = UTF-8
    workgroup = WORKGROUP
    server role = standalone server
    security = user
    passdb backedn = tdbsam
    realm = yohoho.home
    netbios name = Rob
    server string = Rob
    wins support = no
    encrypt passwords = yes
#    interfaces = 192.168.31.250/24 re0 127.0.0.1
#    bind interfaces only = yes
    hosts allow = 192.168.31.

# Enforce a minimum of Samba v2 (Vista / Server 2008) for server and client connections
server min protocol = SMB3
client min protocol = SMB3

# recommended fruit config for MacOS
    vfs objects = catia fruit streams_xattr  
    fruit:posix_rename = yes
    fruit:advertise_fullsync = true
    fruit:metadata = stream
#    fruit:model = MacSamba
    fruit:model = MacPro7,1@ECOLOR=226,226,224
    fruit:copyfile = yes
    fruit:zero_file_id = no
    fruit:veto_appledouble = no
    fruit:nfs_aces = no
    fruit:wipe_intentionally_left_blank_rfork = yes 
    fruit:delete_empty_adfiles = yes 
    fruit:encoding = native
    fruit:aapl = yes
      readdir_attr:aapl_rsize = no
      readdir_attr:aapl_finder_info = no
      readdir_attr:aapl_max_access = no

# (OPTIONAL) Don't allow MacOS clients to write .DS_Store files to server shares
veto files = /.snap/.sujournal/._.DS_Store/.DS_Store/.Trashes/.TemporaryItems/
delete veto files = yes
spotlight backend = tracker

# performance tunning reference 
# https://hilltopsw.com/blog/faster-samba-smb-cifs-share-performance/
# https://fy.blackhats.net.au/blog/2021-03-22-time-machine-on-samba-with-zfs/
# https://serverfault.com/questions/999920/very-slow-smb-speeds-on-macos
# https://gist.github.com/fschiettecatte/02d61e3d36c5f8d36bd45586fc5d0dc7
# https://www.samba.org/~ab/output/htmldocs/Samba3-HOWTO/speed.html
# https://www.dokispace.com/posts/freebsd-file-server/
#
#
#    strict allocate = no
     strict sync = no
     block size = 4096
     disable netbios = yes
     fstype = ZFS
#    read raw = yes
#    write raw = yes
#    strict locking = Auto
#    socket options = TCP_NODELAY IPTOS_LOWDELAY
#    min receivefile size = 32768
    use sendfile = Yes
#    aio max threads = 1000
#    aio read size = 1
#    aio write size = 1
     server smb encrypt = required
# if client has problem of encryption then set to desired

    # Linux supports only kernel oplocks
#    kernel oplocks = yes

#    printing = cups
#    printcap name = cups
#    load printers = yes
#    cups options = raw

## Samba-based Time Machine Backups from https://freebsdfoundation.org/our-work/journal/browser-based-edition/storage-and-filesystems/samba-based-time-machine-backups/
#[TimemMachine]
#comment = Time Machine
#fruit:time machine = yes
#fruit:time machine max size = 1050G
#path = /data/backup/timemachine/%U
#valid users = %U
#browseable = yes
#writeable = yes
#create mask = 0600
#directory mask = 0700
## NOTE: Changing these will require a new initial backup cycle if you already have an existing timemachine share.

#case sensitive = true
#default case = lower
#preserve case = no
#short preserve case = no

[data]
    path = /data
    public = no
    writable = yes
    browsable = yes
    printable = no
    guest ok = no
    veto files = /._*/.DS_Store/
    delete veto files = yes
    valid users = jazzi, @owners
    write list = jazzi
    create mask = 0666
    directory mask = 0755

[music]
    path = /data/music
    public = no
    writable = no
    printable = no
    guest ok = no
    valid users = jazzi

[movie]
    path = /data/movie
    public = yes
    guest ok = yes
    guest user = guest
    writable = no
    printable = no
    map to guest = bad user
```

## MacOS client settings

Here is the settings on MacOS from [ms8.com](https://www.ms8.com/enhancing-macos-smb-stability-and-usability-with-custom-configurations/)

```
cat /etc/nsmb.conf

[default]
# Use NTFS streams if supported
streams=yes

# Soft mount by default
soft=yes

# Disable signing due to macOS bug
signing_required=no

# Disable directory caching
dir_cache_off=yes
dir_cache_async_cnt=0
dir_cache_max_cnt=0
dir_cache_max=0
dir_cache_min=0

# Lock negotiation to SMB2/3 only
# Require SMB2/3
# 7 == 0111  SMB 1/2/3 should be enabled
# 6 == 0110  SMB 2/3 should be enabled
# 4 == 0100  SMB 3 should be enabled
protocol_vers_map=4

# No SMB1, so we disable NetBIOS
# How to disable SMB 1 or NetBIOS in macOS
# https://support.apple.com/en-us/HT211927
# https://support.apple.com/en-us/102050
port445=no_netbios
validate_neg_off=yes

# Turn off notifications
notify_off=yes

# SMB Multichannel behavior
# https://support.apple.com/en-us/102010
mc_on=no
mc_prefer_wired=yes

unix extensions = no
veto files=/._*/.DS_Store/

# Disable SMB session signing
validate_neg_off=yes
```

Also better to disable.DS_Store files as below, 

`sudo defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE`

**Reference:** [https://support.apple.com/de-de/HT208209](https://support.apple.com/de-de/HT208209)

Next step is to disable TCP ACK as below:

`sudo sysctl -w net.inet.tcp.delayed_ack=0` 

**Remember to reboot the MacOS**

## MacOS Finder takes too long time (2+ minutes) to list big directory on SMB share

Many people suggests to disable the preview function on Finder, that might be faster but actually no need to do that. I have lots of pictures on the SMB Share and now I have no problem to list and preview them on MacOS Finder in one second.

You need to put some settings into both server and client.

```
Server Part

[global]
    fruit:aapl = yes
      readdir_attr:aapl_rsize = no
      readdir_attr:aapl_finder_info = no
      readdir_attr:aapl_max_access = no
```

---

```
Client Part

[default]
# Disable directory caching
dir_cache_off=yes
dir_cache_async_cnt=0
dir_cache_max_cnt=0
dir_cache_max=0
dir_cache_min=0
```
