---
layout: post
---

Caddy is a Let's Encrypt TLS default webserver, much easier than Apache or Nginx, I chose it to run Wordpress or Hugo or what's next Nextcloud.

OS: Ubuntu: 22.04
PHP: 7.4
Nextcloud: 25
MariaDB: 

As I already has Caddy2 installed and running, all I need to do is to choose the right version of Nextcloud. The latest Nextcloud requires at least PHP 8.0, so I chose Nextcloud 25.0.

## DNS record

Go to your domain dashboard and set the right IP address for the domain used for nextcloud, such as example.com.

This setting may take over 30 minutes or couple hours to be effective.

## Caddyfile

That's the most complicated part however, just add the following lines into /etc/caddy/Caddyfile should work.

---txt
## For nextcloud

example.com {

    root * /www/nextcloud
    php_fastcgi unix//run/php/php7.4-fpm.sock # for TCP port use php_fastcgi 127.0.0.1:9000
    file_server
    encode gzip

    redir /.well-known/carddav /remote.php/dav 301
    redir /.well-known/caldav /remote.php/dav 301
    redir /.well-known/host-meta /public.php?service=host-meta
    redir /.well-known/host-meta\.json /public.php?service=host-meta-json
        
    # Stop acccess from internet
    @forbidden {
    path /.htaccess
    path /.xml
    path /3rdparty/*
    path /README
    path /config/*
    path /console.php
    path /data/*
    path /db_structure
    path /lib/*
    path /occ
    path /templates/*
    path /tests/*
    }
    respond @forbidden 404

    header {
                Strict-Transport-Security               "max-age=15552000;"
                X-Frame-Options                         "SAMEORIGIN" always
                X-Permitted-Cross-Domain-Policies       "none" always
                X-Robots-Tag                            "none" always
                X-XSS-Protection                        "1; mode=block" always
                X-Content-Type-Options                  "nosniff" always
    }
}

---

## Set up the database

No matter which database used, MariaDB or MySQL, using the commind line to create a databse and a user.

---text
$> mysql -u root -p  # Enter the database
$> CREATE DATABASE nextcloud;
$> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY 'nextcloudPassword';
$> FLUSH PRIVILEGES;
$> \q
---

## Download Nextcloud 25.0

`curl -O https://download.nextcloud.com/server/releases/nextcloud-25.0.9.tar.bz2`

---text
$> cp nextcloud-25.0.9.tar.bz2 /www/
$> tar -xvf nextcloud-25.0.9.tar.bz2
$> sudo chown -R www-data:www-data /www/nextcloud
---

## Restart Caddy

$> sudo systemctl restart caddy.service

If above command failed, then check the log by command `sudo systemctl status caddy` and follow the instructions.

## Visit nextcloud domain and create an administrator user

Open your browser and type the domian name, after that input the nessarary information especially the part about database, the database name and user, as well as the site "localhost".
