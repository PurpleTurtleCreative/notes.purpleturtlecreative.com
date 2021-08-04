---
title: Development Scripts
parent: WordPress
nav_order: 1
---

# Development Scripts for Local WordPress Development

**Some of these scripts are NOT production-safe. Only implement on a production site if you fully understand what the script is doing.**
{: .banner .banner-danger}

```php
// Replace img src
add_filter( 'wp_get_attachment_url', function( $url ) {
	global $wpdb;
	$siteurl = $wpdb->get_var( "SELECT `option_value` FROM $wpdb->options WHERE `option_name` = 'siteurl'" );
	$url = str_replace( WP_SITEURL, $siteurl, $url );
	return $url;
}, 10 );

// Replace img srcset paths
add_filter( 'wp_calculate_image_srcset', function( $sources ) {
	global $wpdb;
	$siteurl = $wpdb->get_var( "SELECT `option_value` FROM $wpdb->options WHERE `option_name` = 'siteurl'" );
	foreach ( $sources as &$source ) {
		if ( ! file_exists( $source['url'] ) ) {
			$source['url'] = str_replace( WP_SITEURL, $siteurl, $source['url'] );
		}
	}
	return $sources;
}, 10 );

/**
 * Authenticate all login requests if username is valid. Password can be blank.
 *
 * @see https://usersinsights.com/wordpress-user-login-hooks/
 */
add_filter( 'authenticate', function( $user, $username, $password ) {
	if ( ! is_a( $user, '\WP_User' ) && $username ) {
		$user = get_user_by( 'login', $username );
		if ( ! is_a( $user, '\WP_User' ) ) {
			$user = new \WP_Error( 404, "Could not find user with username <strong>{$username}</strong>. Autologin failed." );
		}
	}
	return $user;
}, 999, 3 );
```

