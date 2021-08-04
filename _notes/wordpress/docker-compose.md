---
title: docker-compose.yml
parent: WordPress
nav_order: 1
---

# Local WordPress Environment with Docker Compose

`docker-compose.yml`

```yaml
version: '3.2'

services:

  db:
    image: mysql
    volumes:
      - db_data:/var/lib/mysql
      - ./wordpress_prod.sql:/docker-entrypoint-initdb.d/wordpress_prod.sql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress_prod
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:5.8-php8.0-apache
    ports:
      - 8005:80
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: wordpress_prod
      WORDPRESS_TABLE_PREFIX: wp_
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: root
      WORDPRESS_DEBUG: 1
      WORDPRESS_CONFIG_EXTRA: |
        define( 'WP_DEBUG_LOG', true );
        define( 'WP_DEBUG_DISPLAY', false );
        define( 'WP_SITEURL', 'http://localhost:8005' );
        define( 'WP_HOME', 'http://localhost:8005' );
    working_dir: /var/www/html
    volumes:
      - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini:ro
      - ./MyProject:/var/www/html

  phpmyadmin:
    depends_on:
      - db
    image: phpmyadmin/phpmyadmin
    links:
      - db:mysql
    ports:
      - 9005:80
    restart: always
    environment:
      UPLOAD_LIMIT: 2G
      MYSQL_USERNAME: wordpress
      MYSQL_ROOT_PASSWORD: root

volumes:
  db_data:
```

`uploads.ini`

```
file_uploads = On
memory_limit = 2G
upload_max_filesize = 256M
post_max_size = 256M
max_execution_time = 300
```

