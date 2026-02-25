---
layout: post
---

According to the suggestion from [Wordpress Developer Resources](https://developer.wordpress.org/advanced-administration/security/backup/), it's better to backup it before upgrading. Two parts need to be backup:

1. Wordpress Site
2. Wordpress Database

**Do back up the database first, then back up the WordPress Files.**

## Backup MySQL database

[This page](https://geek-docs.com/mysql/mysql-ask-answer/479_mysql_how_to_backup_whole_mysql_database_with_all_users_and_permissions_and_passwords.html) is a good MySQL backup instruction, however I will highlight some commands.

* `mysqldump --all-databases --user=root --password | gzip > backup.sql.gz ## backup whole database including user, permission, password.`
* `gunzip < backup.sql.gz | mysql --user=root --password ## restore it`

In short, [mysqldump](https://dev.mysql.com/doc/refman/8.4/en/mysqldump.html) is a simple yet efficient tool to do backup job.

### Download MySQL backup file with scp

[SCP](https://www.man7.org/linux/man-pages/man1/scp.1.html) is a transfer tool on top of OpenSSH, the simplified use is as below:

`scp SOURCE-FILE DESTINATION-FILE`

So to download the hot backup above from the server to local, a command like below is enough:

`scp username@remote_host:/remote/file.txt /local/directory/`

If your server has SSH Key, also config this server with head name like **server1** in your ~/.ssh/config, then you can just type:

`scp server1:/remote/file.txt ./local/download/`

To download a whole directory, add option -r:

`scp -r server:/remote/directory ./local/download/`

## Backup Wordpress files with rsync

[rsync](https://rsync.samba.org) can do fast increnmental file transfer no matter server to local or local to local. Normally all modern system has this too installed default.

Try command `rsync --version`, if no output then install it.

They way to use **rsync** is as below:

`rsync [option] SRC ... [Dest]

### Caution about the slash ('/') in the end of path

* no slash of source like /data/src will copy the whole directory of **src**
* with slash like /data/src will only copy the files within **src**

### Backup files from server to local

The options are explained very well in [Geek's Blog](https://geek-blogs.com/blog/linux-rsync-examples/#32-远程同步基于-ssh).

`rsync -avz server1:/srv/wordpress ./local/download/ ## option z means compress`
