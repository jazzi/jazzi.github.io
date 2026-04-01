---
layout: post
---

Here are the components:

* Wordpress
* Caddy
* MariaDB
* PHP
* FreeBSD

Here are the steps to go:

1. Install MariaDB and create database and user
2. Install Wordpress and get the configuration job done
3. Install PHP and add two lines into *php.ini* or the database connection will have problem
4. Install Caddy and add new domain into Caddyfile

So hereby goes the details.

---

## MariaDB database

```
pkg install mariadb114-server	# Install it
sysrc mysql-server enable	# Enable it
service mysql-server start	# Start it
mariadb-secure-installation	# Harden it
mariadb -u root -p		# Login then create user/database etc as below
create user wp_user identified by 'new-password';	# create new user
create database wordpress;	# Create new database
grant all on wordpress.* to wp_user;	# Grant permissions
flush frivileges;		# Take effect immediately
quit;				# All MySQL commands end with ;
``` 
## Install Wordpress

```
pkg install wordpress		# Install it
cp /usr/local/www/wordpress/wp-config-sample.php /usr/local/www/wordpress/wp-config.php	# Prepare it
chown -R www:www /usr/local/www/wordpress	# Change ownership
```

## Install PHP

`pkg install phpmailer php82-extension php82-mbstring php82-intl`		# Version might not be 8.2

Actually PHP and the other extensions have been installed along with the Wordpress, so no need to install it again but need some adjustments.

To use php-fpm, edit file */usr/local/etc/php-fpm.d/ww.conf* as below:

Modify

> listen = 127.0.0.1:9000

To

> listen = /var/run/php82.sock

Then, uncomment the following lines:

```
listen.owner = www
listen.group = www
listen.mode = 0660
```

One more step left, or Wordpress would not accept localhost and encounter database connection error.

`cp /usr/local/etc/php.ini-production /usr/local/etc/php.ini`	# Prepare it and add the following two lines

> pdo_mysql.default_socket = /var/run/mysql/mysql.sock
>
> mysqli.default_socket = /var/run/mysql/mysql.sock


```
service php_fpm enable		# Enable it in /etc/rc.conf
service php_fpm start		# Start it
```

## Install Caddy

```
pkg install caddy		# Install it
service caddy enable		# Enable it
```

Then add the configuraitons for your domain into */usr/local/etc/caddy/Caddyfile*

```
teagogo.com, www.teagogo.com {
	root * /usr/local/www/wordpress-zh_CN
	php_fastcgi unix//var/run/php82.sock
	file_server
	encode gzip 
	tls teagogo@qq.com {
		protocols tls1.2 tls1.3
	}


	@disallowed {
		path /xmlrpc.php
		path *.sql
		path /wp-content/uploads/*.php
	}

	rewrite @disallowed '/index.php'
}
```

## Memcached Object Cache

*This is optional as my website seems ok without it.*

Install the following packages to get *Memcached*, or you can try [redis](https://redis.io/downloads/) too.

```
pkg install memcached
pkg install php84-pecl-memcached
```

Then add *memcached_enable="YES"* to /etc/rc.conf, after that you can start it by `service memcached start` and check it by `service memcached status`.

Take a look at the stat of memcached:

`echo "stats settings" | nc localhost 11211`

```
STAT maxbytes 67108864
STAT maxconns 1024
STAT tcpport 11211
STAT udpport 0
STAT inter NULL
STAT verbosity 0
STAT oldest 0
STAT evictions on
STAT domain_socket NULL
STAT umask 700
STAT shutdown_command no
STAT growth_factor 1.25
```

Or play with Telnet as below:

```
$ telnet localhost 11211
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
get foo
VALUE foo 0 2
hi
END
stats
STAT pid 8861
(etc)
```

There is no independent configuration file in FreeBSD for Memcached, but you can set it like below in */etc/rc.conf*:

```
memcached_user = ""
memcached_port = "11211"
memcached_maxconn = "2048"
memcached_cachesize = "4096"
memcached_options = "-l 10.10.1.5"
```

## Cache? Database cache or what?

For small VPN, *Memcached* and *redis* is too heavy, [SQLite Object Cache](https://wordpress.org/plugins/sqlite-object-cache/) is the right option, but in order to get *SQLite Object Cache* plugin installed and working, your OS need to haveSQLite3 exentension to php installed, also better to have another php extension installed too, that's [igbinary](https://www.php.net/manual/en/intro.igbinary.php) or [APCu](https://www.php.net/manual/en/book.apcu.php).

In FreeBSD the three packages are:

* php84-sqlite3
* php84-pecl-APCu
* php84-pecl-igbinary

I chose APCu instead of igbinary, after this package installed, run the following command to get it working:

`service php_fpm restart`

And check **Use APCu** in the plugin setting page.


## Plugins & Themes used for Wordpress eCommerce

* [Advanced Shipment Tracking for WooCommerce](https://wordpress.org/plugins/woo-advanced-shipment-tracking/)
* [Currency Switcher for WooCommerce](https://wordpress.org/plugins/currency-switcher-woocommerce/)
* [Header Footer Code Manager](https://wordpress.org/plugins/header-footer-code-manager/) for adding code 百度统计
* [SimpleTOC](https://wordpress.org/plugins/simpletoc/)
* [Wenprise Alipay Gateway For WooCommerce](https://wordpress.org/plugins/wenprise-alipay-checkout-for-woocommerce/)
* [WooCommerce](https://wordpress.org/plugins/woocommerce/)
* [WooCommerce PayPal Payments](https://wordpress.org/plugins/woocommerce-paypal-payments/)
* [自定义SEO设置]()https://www.jingxialai.com/
* [SQlite3 Object Cache](https://wordpress.org/plugins/sqlite-object-cache/)
* Optional [Smush Image Optimization](https://wordpress.org/plugins/wp-smushit/)
* Optional [WP Super Cache](https://wordpress.org/plugins/wp-super-cache/)

*It's very interesting after both Smush and WP Super Cache deactived, the website feels faster.*

And the theme used is [YITH Wonder](https://wordpress.org/themes/yith-wonder/).

## Summarize up

Nothing different to Linux plantforms, just be aware of the *php-fpm* who has a different location on FreeBSD, also the damon *php_fpm* is not *php-fpm*, it is underline.

Next better practise would be jail up them.

Many thanks to the following articles:

1. https://forums.freebsd.org/threads/wordpress-install.85573/
2. https://it-notes.dragas.net/2022/07/18/freebsd-caddy-and-php-a-perfect-match/
