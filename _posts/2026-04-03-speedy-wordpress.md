---
layout: post
---

Two ways to speedy Wordpress website: Cache and Adjustment of configuration

**Cache**:

1. PHP level: Zend OpCache and PHP JIT
2. Database level: SQLite Object Cache, Memcached and Redis
3. Page level: WP Super Cache plugin
4. Proxy level: Varnish, nginx proxy or Caddy handler-cache

**Adjustment**:
1. PHP pm model: static, dynamic and ondemand(good for small website)
2. Disable PHP gc: zend.enable_gc = Off

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



## Index MariaDB

```
mysql -u root -p ## login MariaDB
SHOW DATABASES; ## list all databases
USE wordpress; ## change to this database
SHOW TABLES; ## list all tables and notice the wp_ or wpp_
```

After that you can run the following commands to index the database:

```
-- 1. 首页产品+博客列表（覆盖 post_status, post_type, post_date）
ALTER TABLE wpp_posts ADD INDEX idx_status_type_date (post_type, post_status, post_date);

-- 2. WooCommerce 配置读取（高频小查询，容易被忽略）
ALTER TABLE wpp_options ADD INDEX idx_option_name (autoload);

-- 3. 产品价格与库存查询（WooCommerce 最大瓶颈）
ALTER TABLE wpp_postmeta ADD INDEX idx_post_id_meta_key (post_id, meta_key(191));

-- 4. 订单项查询（WooCommerce 订单详情页核心）
ALTER TABLE wpp_woocommerce_order_items ADD INDEX idx_item_type_order (order_item_type, order_id);

-- 5. 评论（如果评论数量很少可以不用）
ALTER TABLE wpp_comments ADD INDEX idx_approved_post (comment_approved, comment_post_ID);

---

**执行建议**：
1. Backup database: instruction is [here](/backup-wordpress-with-mysqldump-and-rsync.html)
2. Replace the prefix of table: something like wp_
3. Verify: command `SHOW INDEX FROM wp_posts;`

Before you do all these might you find out what slows it down, plugin *Query Monitor* can help or enable *slow_query_log* in configurations and analysis it by command `mysqldumpslow` every week.

**TIPS**: There is an interesting command that can analysis *the slow SQL* from *Query Monitor* and find out why, this command looks like `EXPLAIN [你刚才复制的SQL语句];`

---

## Helpfuls links

1. [Redis vs Memcached vs APCu: Best WordPress Object Cache](https://uxnitro.com/redis-vs-memcached-vs-apcu-best-wordpress-object-cache/)
2. [Object Cache | Page Cache | OP Cache – Caching for WordPress](https://immanuelraj.dev/wordpress-caching-opcache-object-page-optimization/)
3. [MariaDB 数据库之索引详解](https://www.cnblogs.com/LyShark/p/10189504.html)
4. [高性能WordPress站优化技巧](https://www.zhichai.top/archives/4643)
5. [Caddy额外缓存配置,加快网站访问速度](https://blog.wjqserver.com/post/caddy-cache/)






