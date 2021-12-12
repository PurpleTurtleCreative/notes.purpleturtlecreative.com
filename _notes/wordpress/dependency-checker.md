---
title: Dependency Checker
parent: WordPress
nav_order: 1
---

# Dependency Checker

Require certain conditions to be met for plugin activation. Otherwise, display a notice on error.

*ptc-plugin.php*

```php
if ( is_admin() ) {
	/* Register Admin-Only Functionality */
	foreach ( glob( PLUGIN_PATH . 'classes/admin/*.php' ) as $file ) {
		require_once $file;
	}

	Dependency_Checker::register();

	if ( false === Dependency_Checker::has_failures() ) {
		// Only execute code if dependency checks pass.
		Admin_Pages::register();
		Enqueue_Admin::register();
	}
}
```

*class-dependency-checker.php*

```php
<?php
/**
 * Handles activation requirements.
 *
 * @link https://wordpress.stackexchange.com/a/131447 Original inspiration on
 * this implementation.
 */

namespace PTC_Plugin;

defined( 'ABSPATH' ) || die();

class Dependency_Checker {

	public static function register() {
		add_action( 'admin_init', [ __CLASS__, 'maybe_deactivate_with_notice' ], 5 );
	}

	public static function has_failures() {
		if (
			function_exists( '\acf_add_options_sub_page' )
			&& function_exists( '\acf_add_local_field_group' )
			&& function_exists( '\get_field' )
		) {
			return false;
		}

		return true;
	}

	public static function maybe_deactivate_with_notice() {
		if ( Dependency_Checker::has_failures() ) {
			// Notify the user that activation failed.
			add_action( 'admin_notices', [ __CLASS__, 'display_failure_notice' ] );
			// Deactivate this plugin since installation requirements were not met.
			deactivate_plugins( PLUGIN_BASENAME );
			// Do not display success notice saying this plugin was activated.
			if ( isset( $_GET['activate'] ) ) {
				unset( $_GET['activate'] );
			}
		}
	}

	public static function display_failure_notice() {
		?>
		<div class="notice notice-error">
			<p><strong>This plugin requires ACF Pro to be installed and activated.</strong> The plugin has been deactivated.</p>
		</div>
		<?php
	}
}
```

