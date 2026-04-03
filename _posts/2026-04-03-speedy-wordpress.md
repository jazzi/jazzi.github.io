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
