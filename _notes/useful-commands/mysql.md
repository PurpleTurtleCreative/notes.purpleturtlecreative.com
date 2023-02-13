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

## Inspecting a Runaway Process

When there’s a runaway MySQL query (confirmed via `htop` in SSH), log into `mysql -uxxxxx -pxxxxx` and then run the query [`SHOW FULL PROCESSLIST;`](https://dev.mysql.com/doc/refman/8.0/en/show-processlist.html) to see which query is currently running (ie. hung).
