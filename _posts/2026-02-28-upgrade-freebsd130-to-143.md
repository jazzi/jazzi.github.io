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

## upgrade mariadb from 11.4 to 11.8

Actually I got very nice instructions from [Upgrade MariaDB from 10.6 LTS to 11.8 LTS](https://www.gaelanlloyd.com/blog/upgrade-mariadb-106-to-118/), hereby is the key steps:

### backup in advance

`mysqldump --all-databases > /root/mysql-upgrade-backup-$(date +%Y%m%d).sql`

`tar -cvzf /root/mysql_config_backup_$(date +%Y%m%d).tar.gz /usr/local/etc/mysql`

Then stop service and install it:

```
service mysql-server stop
pkg install mariadb118-server
```

Then start it again:

`service mysql-server start`

`mariadb-upgrade -u root -p`

*There is no need to remove the old MariaDB packages as it will be removed automatically.*

### error with mariadb-upgrade

There is a very frequent error as below:

> ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)

As the correct way to run this command:

Before

`mariadb-upgrade`

After (Correct way)

`mariadb-upgrade -u root -p`

For more errors, check [this post](https://runebook.dev/en/docs/mariadb/mariadb-upgrade/index).

## /etc/master.passwd

When upgrade from 13.2 to 14.0, there is a big change that need you pay a close look and edit it, as the default shell for root changed from csh to sh in *FreeBSD 14*. 

You'd better to change the shell of root from */usr/bin/csh* to */usr/bin/sh* and keep your existing part still as below:

> root:$900$2dwsWdTSXoA/.pz2$n1zXjcF9kO6ERUooHHcYZWmtb/HwtaR4xU5MY48VDisYAvypYGCbHI6ietZXFlkoRGnpeUYIG6qXN1OD4q.uS60:0:0::0:0:Charlie &:/root:/bin/sh

## Upgrade PHP82 to PHP84

According to this excellent post - [Upgrade PHP 8.x on FreeBSD](https://www.gaelanlloyd.com/blog/how-to-upgrade-php-80-to-82-on-freebsd/), all I need to do is just to install new packages and it will remove old ones and upgrade automatically.

`pkg query -a %n | grep php8`

Above command will list all php packages installed, then install new ones accordingly as below:

`# pkg install php84 php84-ctype php84-curl php84-dom php84-exif php84-extensions php84-fileinfo php84-filter php84-ftp php84-gd php84-icon php84-intl php84-mbstring php84-mysqli php84-opcache php84-pdo  php84-pdo_sqlite php84-pecl-imagick php84-phar php84-posix php84-session php84-simplexml php84-sqlite3 php84-tokenizer php84-xml php84-xmlreader php84-xmlwriter php84-zip php84-zlib`

Then edit file /usr/local/etc/php-fpm.d/www.conf and change *php82.sock* to *php84.sock* as below:

```
listen = /var/run/php84.sock
```

As Web Server Caddy is used here, so we need to modiy the Caddy settings too.

`vi /usr/local/etc/caddy/Caddyfile`

```
php_fastcgi unix//var/run/php84.sock
```

Then restart php-fpm and Caddy as below:

`service php_fpm restart`

`service caddy restart`
