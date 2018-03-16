---
layout: post
---
This is my note about how to build a fast & secure Wordpress eCommerce website with WooCommerce & StoreFront theme.

# Install softwares

## Install LAMP on VPS

### Get Linux ready
1. There are many options, finally I chose Debian 9 as it consumes less memory than Ubuntu, and I don't know much about CentOS, just grab the tool I am good at.

2. The next thing is about SSH, about how to remotely control this OS safely. I edit /etc/ssh/ssd_config to abdon login through password but RSA only (ssh-copy-id -i ~/.ssh/id_rsa.pub "root@192.1.8.8 -p 60021").

    PasswordAuthentication  no
    RSAAuthentication       yes
    PubkeyAuthentication    yes

3. Turn on BBR to increase internet speed, as Debian 9 already shipped with Kernel 4.9, so can just turn it on.

    echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
    echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
    sysctl -p    //save the changes

Then double check the status, 'sysctl net.ipv4.tcp_available_congestion_control', if something like below show out means you got it.

> net.ipv4.tcp_available_congestion_control = bbr cubic reno 

Then 'lsmod | grep bbr' to see if **tcp_bbr** is there.

### Install LAMP pack
I chose [Teddysun's LAMP](https://github.com/teddysun/lamp) for converience, you can download it there or [here](https://lamp.sh/download.html).

Instead I use git to download & keep updated, 'git pull https://github.com/teddysun/lamp.git'

This LAMP pack include automatically deployment of SSL from [Let's Encrypt](https://letsencrypt.org) which is free and excellent. It can also renew the certificate by 'crond', type 'crontab -l' and make sure the following line is there.

> 0 3 */7 * * /bin/certbot renew --disable-hook-validation --renew-hook "/etc/init.d/httpd restart"

**I choose PHP 7.1 with imagick on & MariaDB & Apache** 

#### Some adjustments on MariaDB to speed up Wordpress
Edit MariaDB configuration */etc/my.cnf*

    default-storage-engine = MyISAM
    key-buffer-size = 256M
    query-cache-type = 1  ## turn query cache on
    query-cache-size = 256M
    query-cache-limit = 2M   ## cache will stop if more than 2M

#### Some adjustments on PHP
Edit /usr/local/php/php.d/php.ini, for more details, check [here](http://blog.csdn.net/weixin_36333654/article/details/52770325)

    opcache.enable=1
    opcache.enable_cli=1

#### There is also some lines about how to adjust Nginx, check [here](http://www.elecfans.com/d/633003.html)

#### Add a Virtual Host on Apache, be sure to edit your DNS and mapping the domain to the IP before this step, as the auto SSL certificate apply gotta need this.

Type following commands to add a virtual host:

    lamp add      // 创建虚拟主机
    lamp del      //删除虚拟主机
    lamp list     //列出虚拟主机

**For security good, be sure to change the prefix from *wp* to others like in database table**

## Install Wordpress with WooCommerce & Storefront theme

1. Download the latest WP and root login VPS

2. Upload WP files to /data/www/newdomain.com/

> ssh-copy-id -i ~/wordpress.tar.gz “root@192.1.8.8 -p 60021"

Then decompress it:

> tar -jvf wordpress.tar.gz 

3. After successfully installed, login and change the URL & SiteURL into https in *Settings* -> *General*, or later the WooCommerce status might report https not turn on, if you meet this situation, here is [the solution](https://github.com/woocommerce/woocommerce/issues/13921).

4. Add following lines above where it says /* That's all, stop editing! Happy blogging. */ into wp-config.php to [Allow MultiSite](https://codex.wordpress.org/Create_A_Network).

> define('WP_ALLOW_MULTISITE', true);

5. For eCommerce, I installed theme named **Storefront** which will fetch WooCommerce plugin automatically.

6. Other plugins I installed: Language Redirect & WP Super Cache & WP Live Chat Support 


### Secure Wordpress ###

* delete readme.html to prohibit showing the world your WP virsion

* set the rights

    find /your/wordpress/folder/ -type d -exec chmod 755 {} \;
    find /your/wordpress/folder/ -type f -exec chmod 644 {} \;

* add line below into the head of .htaccess to prohibit people visiting your WP directory

* replace the security [key generate](http://www.luoxiao123.cn/go/?url=https://api.wordpress.org/secret-key/1.1/salt/)d by wp from wp-config.php

For more suggestions on Wordpress security, check [here](http://www.luoxiao123.cn/1172-2.html)
