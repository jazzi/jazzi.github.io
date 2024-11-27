---
layout: post
---

In order to change the site URL of Wordpress, I prefer to do it with command line as below:

Firstly edit *wp-config.php* file and change the following two lines

```
define( 'WP_SITEURL', 'https://nothing.com' );
define( 'WP_HOME', 'https://nothing.com' );
```

Secondly issue the following four mySQL commands

```
UPDATE wp_options SET option_value = replace(option_value, 'https://old.example.com', 'https://new.example.com') WHERE option_name = 'home' OR option_name = 'siteurl';

UPDATE wp_posts SET guid = replace(guid, 'https://old.example.com', 'https://new.example.com');

UPDATE wp_posts SET post_content = replace(post_content, 'https://old.example.com', 'https://new.example.com');

UPDATE wp_postmeta SET meta_value = replace(meta_value, 'https://old.example.com', 'https://new.example.com');
```

But before doing that you need to 

* login to MySQL `mysql -u root -p`, 
* list databases `SHOW DATABASES;`
* change database `USE database_name_you_want;`

---

Sure before new URL available, you need to change the http server too, also point the right IP address to new domain through your domain vendor. 

*Lastly, login to Wordpress dashboard and delete the cache if you have cache plugin installed.*
