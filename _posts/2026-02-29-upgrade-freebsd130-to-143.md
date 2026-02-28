---
layout: post
---

To upgrade FreeBSD from 13.0 to 14.3, there is a need to upgrade to 13.2 in advance as there is an important patch according to [some nice comments on Reddit](https://www.reddit.com/r/freebsd/comments/1h7gsi8/upgrade_path/), and I can prove it. 

I upgraded from 13.0 -> 13.1 -> 13.2 -> 13.5 -> 14.0 -> 14.3

Hereby is the commands:

```
freebsd-update fetch
freebsd-update install
freebsd-update -r 13.2-RELEASE upgrade
freebsd-update install

## then do a reboot
shutdown -r now
freebsd-update install
## then better another reboot and upgrade packages
shutdown -r now
pkg-static upgrade -f

## finally a install and reboot again
freebsd-update install
shutdown -r now
```

Still something happened and the solution reserved to be marked.

## can not login through ssh

This is because the new configuration file merged with old one and you need to edit it manually, if you do not edit it, the daemon *sshd* will refuse to start thus can not accept remote login.

Remove the followings:

> ====
> >>>>>
> <<<<<

And correct the following line according to the output of command `ssh -Q cipher`

> Ciphers 3des-cbc,aes128-cbc,aes192-cbc,aes256-cbc,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
 
## caddy server can not start after upgrade

```
# service caddy start
Starting caddy... Error: Caddy failed to start
Check the caddy log: /var/log/caddy/caddy.log
```

So I take a look at the log file
```
Error: loading initial config: loading new config: http app module: start: listening on :443: listen tcp :443: bind: permission denied
Error: caddy process exited with error: exit status 1
```

The reason is about non-privileged user can not open port 443, we need to add a package named *portacl-rc*:

```
# pkg install security/portacl-rc
# sysrc portacl_users+=www
# sysrc portacl_user_www_tcp="http https"
# sysrc portacl_user_www_udp="https"
# service portacl enable
# service portacl start
```

Then do some config jobs:

```
# service caddy enable # 请按顺序执行
# service caddy start # 请按顺序执行
# service caddy stop # 请按顺序执行
# sysrc caddy_user=www caddy_group=www
# chown -R www:www /var/db/caddy /var/log/caddy /var/run/caddy
```

