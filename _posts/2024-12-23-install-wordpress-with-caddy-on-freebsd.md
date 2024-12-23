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

`pkg install phpmailer php82-extension`		# Version might not be 8.2

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

## Summarize up

Nothing different to Linux plantforms, just be aware of the *php-fpm* who has a different location on FreeBSD, also the damon *php_fpm* is not *php-fpm*, it is underline.

Next better practise would be jail up them.

Many thanks to the following articles:

1. https://forums.freebsd.org/threads/wordpress-install.85573/
2. https://it-notes.dragas.net/2022/07/18/freebsd-caddy-and-php-a-perfect-match/
