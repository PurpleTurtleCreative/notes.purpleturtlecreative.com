---
title: Import Large Database
parent: Docker
nav_order: 1
---

# Import Large (2GB+) MySQL Dump into Docker Container

## Copy SQL file to container
```bash
% docker cp wordpress_prod.sql wordpress_db_1:/home
```

## Log into Docker MySQL container
```bash
% docker exec -it wordpress_db_1 bash
% mysql -uroot -proot
```

## Import MySQL data from copied file
Source: [https://stackoverflow.com/a/22155778](https://stackoverflow.com/a/22155778)
```mysql
mysql> set global net_buffer_length=1000000; --Set network buffer length to a large byte number
mysql> set global max_allowed_packet=1000000000; --Set maximum allowed packet size to a large byte number
mysql> SET foreign_key_checks = 0; --Disable foreign key checking to avoid delays,errors and unwanted behaviour
mysql> source /home/wordpress_prod.sql --Import your sql dump file
mysql> SET foreign_key_checks = 1; --Remember to enable foreign key checks when procedure is complete!
```

In fewer copy-paste commands:
```
set global net_buffer_length=1000000; set global max_allowed_packet=1000000000; set foreign_key_checks = 0;
source /home/wordpress_prod.sql
set foreign_key_checks = 1;
```