I removed all contents in my previous VPS which had Apache/MySQL/PHP/Wordpress/Varnish installed, the reason I put this trigger is because it's too complex and now the ONLY function needed is to serve static file website which is generated by [Hugo](https://gohugo.io).

## Why Caddy 2.0?

> Caddy 2 is a powerful, enterprise-ready, open source web server with automatic HTTPS written in Go

That's the YELLO on [Caddy](https://caddyserver.com) website, to me:

1. Written in Go (that's something to me, as Hugo is written in Go too)
2. Automatic https ready (no hassel with ssl now yeah)
3. Simple setting (compare Nginx/Apache, the configuration is much much easier)
4. Fast (almost as quick as Nginx, it's light and powerful)
5. Free to use even for commercial (Caddy version 1 has restriction on _commercial use_ but not Caddy 2, also it's open source)

## Why not Apache or Nginx?

Actually Apache has been serving me for over 2 years, it's quite stable, just one problem, you have to right set it or it would consume too much resources and blow your server down especially RAM < 2G. Although it works well for all these time but there is always a worry sometime it will be down, it's not about what happen but about what might happen, that's a big burden to daily life.

"Then you can choose Nginx." I hear your voice, that's right, Nginx is a good option for heavy load server, I did the home work and almost transfer to it. And you know what freaks me out? The complex SSL/HTTPS settings and not-so-easy configuration. [Let's Encrypt](https://letsencrypt.org) has become the _must_ and _should_ have for every web server becasue of it's high quality and free use. In 2021 most of website are SSL secured like tap water for civilized life, so why do I still need to hack it by myself? This may make sense for big project but means nothing for my small site, I just need it work.

## Secure VPS

I do not want to be alarmed "Dooom your website is hacked!" The internet is something like the jungle wild forest, bad guy or naughty boy are there, no matter for fun or evil, they will doom your day. Here are some basic treatments to harden your server:

1. Create a common user and let it do the work instead of User _Root_ (adduser, sudo)
2. Upgrade system regulately (yum upgrade)
3. Forbid SSH login through _password_ but ssh key and no _Root_ login (vim /etc/ssh/sshd_config)
4. Change SSH default port 22 to something else like 12306
5. Set up a [firewall](https://docs.fedoraproject.org/en-US/quick-docs/firewalld/) and open port for SSH and https (firewall-cmd --permanent --zone=public --add-port=12306/tcp, firewall-cmd --permanent --zone=public --add-port=80/tcp )
6. [Fail2ban](https://www.linuxcapable.com/how-to-install-fail2ban-with-firewalld-on-rocky-linux-8/), install package fail2ban and fail2ban-firewalld, then add following text into /etc/fail2ban/jail.local
```
[sshd]
enabled = true

#Permanent ban by setting a value of -1
bantime = -1
```
Remember, you can ban one IP by `sudo fail2ban-client set sshd banip <ip address>`

7. Set right [permission](https://www.cnblogs.com/sochishun/p/7413572.html) for files and directory:
```
sudo chown -R RegularUser:caddy /srv/www
sudo chmod -R 750 /srv/www ## for directory
sudo chmod -R 640 /srv/www ## for files
```
## Caddyfile

Okay, finally the show comes, the [Caddyfile](https://shibumi.dev/posts/new-caddyfile-and-more/).

```
# This rule matches on www.nullday.de and www.nspawn.org and strips off the www
# part. I need this for Hugo (my static website generator). Otherwise Hugo will
# generate wrong sitemap.xml files. You might ask your self what {http.request.host.labels.1} mean.
# These are templates. This way you can access various internal Caddy variables.
# For example the host name in the incoming HTTP request.

www.nullday.de, www.nspawn.org, www.shibumi.dev {
	redir * https://{http.request.host.labels.1}.{http.request.host.labels.0}{path}
}

# This part is the actual server configuration. I match on my domains nullday.de and nnspawn.org.
# First I activate the file_server for serving static files.

nspawn.org, shibumi.dev {
	file_server
	root * /srv/www/{host}/public/
	# And here I set all headers, that Caddy doesn't set on default.
        # Public-Key-Pins is not set here as it's not recommended any more.
	# TLS settings are not necessary, because Caddy has strong TLS defaults.
	header {
		Strict-Transport-Security "max-age=31536000; includeSubDomains; preload; always"
		X-Frame-Options "SAMEORIGIN"
		X-Content-Type-Options "nosniff"
		X-XSS-Protection "1; mode=block"
		Content-Security-Policy "default-src 'none'; base-uri 'self'; form-action 'none'; img-src 'self'; script-src 'self'; style-src 'self'; font-src 'self'; worker-src 'self'; object-src 'self'; media-src 'self'; frame-ancestors 'none'; manifest-src 'self'; connect-src 'self'"
		Referrer-Policy "strict-origin"
		Feature-Policy "geolocation 'none';midi 'none'; sync-xhr 'none';microphone 'none';camera 'none';magnetometer 'none';gyroscope 'none';speaker 'none';fullscreen 'self';payment 'none';"
		Expect-CT "max-age=604800"
	}
	# Lastly we just enable zstd and gzip compression.
	# Note: zstd compression is not yet supported by browsers.
	# You may ask yourself why I don't enable brotli.
	# The brotli impementation in caddy performs surprisingly bad.
	# I hope the caddy devs are going to fix this..
	encode {
		zstd
		gzip
	}
}

```
## Deploy Hugo files to webserver

There are many ways to [deploy Hugo](https://gohugo.io/hosting-and-deployment/deployment-with-rsync/) files, I chose Rsync, so be sure to install it on both local and remote server.

So create a bash file to get job automatic done.

```
$: vim deploy

#!/bin/zsh
USER=who0areyou
HOST=192.168.1.0
DIR=/srv/www/shibumi.dev/public/   # might sometimes be empty!

hugo && rsync -avz --delete public/ ${USER}@${HOST}:${DIR}

exit 0
```
 


