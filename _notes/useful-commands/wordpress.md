---
title: WordPress
parent: Useful Commands
nav_order: 1
---

# Useful Commands: WordPress

## Robots: noindex, nofollow
Good for staging environments so they won't be indexed by search engines.
```php
function robots_noindex( $robots ) {
	$robots['noindex'] = true;
	$robots['nofollow'] = true;
	return $robots;
}
add_filter( 'wp_robots', 'robots_noindex', 999, 1 );
```

## Gotcha: `get_posts()` of 'any' `post_type`
The documentation for [WordPress's post query arguments](https://developer.wordpress.org/reference/classes/wp_query/#post-type-parameters) says:

> ‘any‘ – retrieves any type **except revisions and types with ‘exclude_from_search’ set to true**

```php
public static function get_post_ids() {

	$post_ids = get_posts(
		array(
			'fields'           => 'ids',
			'post_type'        => get_post_types(), // 'any' does not include hidden post types.
			'suppress_filters' => true,
			'meta_key'         => static::META_KEY,
			'meta_value'       => '1',
			'posts_per_page'   => -1, // do not limit results.
		)
	);

	return is_array( $post_ids ) ? $post_ids : array();
}
```