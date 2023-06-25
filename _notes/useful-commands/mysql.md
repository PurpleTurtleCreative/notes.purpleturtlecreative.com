---
title: MySQL
parent: Useful Commands
nav_order: 1
---

# Useful Commands: MySQL

## UPDATE postmeta value WHERE other postmeta keys don't exist

Must be done in a single SQL transaction in PHPMyAdmin. Use the first two queries to set a variable containing the postmeta records IDs to be affected, and then use the last `SELECT` to preview or `UPDATE` to perform the update.

We use `GROUP_CONCAT` and a `@variable` to avoid the MySQL error about updating a table that is used in the subquery's `FROM`.

```sql
SET session group_concat_max_len=15000;

SELECT
    GROUP_CONCAT(prefix.`meta_id`)
INTO
    @prefix_meta_ids
FROM
    `wp_postmeta` prefix,
    `wp_posts` posts
WHERE
    NOT EXISTS (
        SELECT *
        FROM `wp_postmeta` fname
        WHERE
            fname.`post_id` = prefix.`post_id`
            AND fname.`meta_key` = 'first_name'
    )
    AND NOT EXISTS (
        SELECT *
        FROM `wp_postmeta` lname
        WHERE
            lname.`post_id` = prefix.`post_id`
            AND lname.`meta_key` = 'last_name'
    )
    AND prefix.`meta_key` = 'prefix'
    AND prefix.`meta_value` != ''
    AND prefix.`meta_value` IS NOT NULL
    AND posts.`ID` = prefix.`post_id`
    AND posts.`post_type` = 'company';

## To preview affected records, use this:

SELECT
    prefix.*
FROM
    `wp_postmeta` prefix
WHERE
    FIND_IN_SET(prefix.`meta_id`, @prefix_meta_ids);

## To perform the update, use this:

UPDATE
    `wp_postmeta` prefix
SET
    prefix.`meta_value` = ''
WHERE
    FIND_IN_SET(prefix.`meta_id`, @prefix_meta_ids);
```

## Resolve WooCommerce orders customer user ID

```sql
# MySQL query to correct _customer_user user ID shop_order
# meta values by _billing_email matching user emails.
#
# :warning: Change table names before running :bangbang:

UPDATE `wp_postmeta` customer_user
JOIN `wp_postmeta` billing_email
	ON billing_email.`post_id` = customer_user.`post_id`
JOIN `wp_users` user
	ON billing_email.`meta_value` = user.`user_email`
SET customer_user.`meta_value` = user.`ID`
WHERE billing_email.`meta_key` = '_billing_email'
  AND customer_user.`meta_key` = '_customer_user'
```

## Delete postmeta values where not already empty

Useful for clearing a settings field across many posts.

```sql
# Delete postmeta values where not already empty.

SELECT * FROM utb_2_postmeta postmeta
INNER JOIN utb_2_posts posts
  ON posts.ID = postmeta.post_id
WHERE
  posts.post_type = 'sfwd-courses'
  AND postmeta.meta_key = '_is4wp_membership_levels'
  AND postmeta.meta_value <> ''

UPDATE utb_2_postmeta postmeta
INNER JOIN utb_2_posts posts
  ON posts.ID = postmeta.post_id
SET postmeta.meta_value = ''
WHERE
  posts.post_type = 'sfwd-courses'
  AND postmeta.meta_key = '_is4wp_membership_levels'
```

## WooCommerce Product Counts

Useful for marketing and "social proof" purposes.

```sql
# Count line item instances of 12345 purchased.

Select
	Count(wwoi.meta_id) As Sum_meta_id
From
	wp_posts wppo
	Inner Join
		wp_woocommerce_order_items wwoi1 On wppo.ID = wwoi1.order_id
	Inner Join
		wp_woocommerce_order_itemmeta wwoi On wwoi.order_item_id = wwoi1.order_item_id
Where
	wppo.post_type = 'shop_order' And
	( wppo.post_status = 'wc-completed' Or
	wppo.post_status = 'wc-processing' ) And
	wwoi.meta_key = '_product_id' And
	wwoi.meta_value = 12345

# Count total quantity of 12345 purchased.

Select
	SUM(wwoi_qty.meta_value)
From
	wp_posts wppo
	Inner Join
		wp_woocommerce_order_items wwoi1 On wppo.ID = wwoi1.order_id
	Inner Join
		wp_woocommerce_order_itemmeta wwoi On wwoi.order_item_id = wwoi1.order_item_id
	Inner Join
		wp_woocommerce_order_itemmeta wwoi_qty On wwoi_qty.order_item_id = wwoi1.order_item_id
Where
	wppo.post_type = 'shop_order' And
	( wppo.post_status = 'wc-completed' Or
	wppo.post_status = 'wc-processing' ) And
	wwoi.meta_key = '_product_id' And
	wwoi.meta_value = 12345 And
	wwoi_qty.meta_key = '_qty'

# Count unique customer billing emails for orders containing 12345.

Select
	COUNT(DISTINCT ordermeta.meta_value)
From
	wp_posts wppo
	Inner Join
		wp_woocommerce_order_items wwoi1 On wppo.ID = wwoi1.order_id
	Inner Join
		wp_woocommerce_order_itemmeta wwoi On wwoi.order_item_id = wwoi1.order_item_id
	Inner Join
		wp_postmeta ordermeta On wppo.ID = ordermeta.post_id
Where
	wppo.post_type = 'shop_order' And
	( wppo.post_status = 'wc-completed' Or
	wppo.post_status = 'wc-processing' ) And
	wwoi.meta_key = '_product_id' And
	wwoi.meta_value = 12345 And
	ordermeta.meta_key = '_billing_email'

# Count orders containing 12345 that is NOT a bundled item.

Select
	COUNT(DISTINCT wppo.ID)
From
	wp_posts wppo
	Inner Join
		wp_woocommerce_order_items wwoi1 On wppo.ID = wwoi1.order_id
	Inner Join
		wp_woocommerce_order_itemmeta wwoi On wwoi.order_item_id = wwoi1.order_item_id
Where
	wppo.post_type = 'shop_order' And
	( wppo.post_status = 'wc-completed' Or
	wppo.post_status = 'wc-processing' ) And
	wwoi.meta_key = '_product_id' And
	wwoi.meta_value = 12345 And
	wwoi.order_item_id NOT IN (
		SELECT order_item_id
		FROM wp_woocommerce_order_itemmeta
		WHERE meta_key = '_bundled_item_id'
	)
```

## Inspecting a Runaway Process

When there’s a runaway MySQL query (confirmed via `htop` in SSH), log into `mysql -uxxxxx -pxxxxx` and then run the query [`SHOW FULL PROCESSLIST;`](https://dev.mysql.com/doc/refman/8.0/en/show-processlist.html) to see which query is currently running (ie. hung).

## MySQL InnoDB Optimization

For databases running queries with large query results, it's important to increase the buffer pool so the cache remains effective:

- [https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool.html)

- [https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-resize.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-resize.html)

Display the current InnoDB buffer pool size in gigabytes:

```sql
SELECT @@innodb_buffer_pool_size/1024/1024/1024;
```

Display all of the MySQL daemon's options and their values:

```bash
mysqld --verbose --help
```

Review and update the InnoDB configuration variables:

- [https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html)

- [https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html)

```sql
SHOW VARIABLES;
SET innodb_page_cleaners = 4, innodb_buffer_pool_instances = 8,innodb_buffer_pool_size = 23622320128;
```

## Connecting to an External Database

```bash
mysql --host="12.345.6.7" --user="mydbuser" --password="mydbpass"
```

Create database droplet in DigitalOcean. Use internal, private IP address to connect (such as logging in via CLI or updating wp-config.php).