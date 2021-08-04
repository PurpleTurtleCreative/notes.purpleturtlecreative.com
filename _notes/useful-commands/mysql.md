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

