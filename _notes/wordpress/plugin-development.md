---
title: Plugin Development
parent: WordPress
nav_order: 1
---

# Plugin Development

Common scripts and file structures for WordPress plugin development.

## Main Plugin File

The foundation for a new WordPress plugin:

```php
<?php
/**
 * WordPress Plugin
 *
 * @author            Michelle Blanchette
 * @copyright         2022 Michelle Blanchette
 * @license           GPL-3.0-or-later
 *
 * @wordpress-plugin
 * Plugin Name:       WordPress Plugin
 * Description:       
 * Version:           1.0.0
 * Requires PHP:      
 * Requires at least: 
 * Tested up to:      
 * Author:            Purple Turtle Creative
 * Author URI:        https://purpleturtlecreative.com/
 * License:           GPL v3 or later
 * License URI:       https://www.gnu.org/licenses/gpl-3.0.txt
 */

/*
This program is open-source software: you can redistribute it and/or modify
it UNDER THE TERMS of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see https://www.gnu.org/licenses/gpl-3.0.txt.
*/

namespace PTC_Plugin;// @TODO: Change namespace.

defined( 'ABSPATH' ) || die();

/**
 * The absolute path to this plugin's main file.
 *
 * @since 1.0.0
 *
 * @var string PLUGIN_FILE
 */
define( __NAMESPACE__ . '\PLUGIN_FILE', __FILE__ );

/**
 * The absolute path to this plugin's directory, ending with a slash.
 *
 * @since 1.0.0
 *
 * @var string PLUGIN_PATH
 */
define( __NAMESPACE__ . '\PLUGIN_PATH', plugin_dir_path( __FILE__ ) );

/**
 * The full url to this plugin's directory, ending with a slash.
 *
 * @since 1.0.0
 *
 * @var string PLUGIN_URL
 */
define( __NAMESPACE__ . '\PLUGIN_URL', plugins_url( '/', __FILE__ ) );

/**
 * This plugin's current version, as defined in the plugin header.
 *
 * @since 1.0.0
 *
 * @var string PLUGIN_VERSION
 */
define( __NAMESPACE__ . '\PLUGIN_VERSION', get_file_data( __FILE__, [ 'Version' => 'Version' ], 'plugin')['Version'] ?? '0.0.0' );

/**
 * This plugin's basename, like "my-plugin/my-plugin.php".
 *
 * @since 1.0.0
 *
 * @var string PLUGIN_BASENAME
 */
define( __NAMESPACE__ . '\PLUGIN_BASENAME', plugin_basename( __FILE__ ) );

/**
 * This plugin's directory basename, like "my-plugin".
 *
 * @since 1.0.0
 *
 * @var string PLUGIN_SLUG
 */
define( __NAMESPACE__ . '\PLUGIN_SLUG', dirname( PLUGIN_BASENAME ) );

/**
 * The namespace for all v1 REST API routes registered by this plugin.
 *
 * @since 1.0.0
 *
 * @var string REST_API_NAMESPACE_V1
 */
define( __NAMESPACE__ . '\REST_API_NAMESPACE_V1', 'ptc-resources/v1' );

/* CODE REGISTRATION */

/**
 * Requires a class file and calls its static "register" method.
 *
 * Class file names and class names must follow WordPress naming conventions.
 * For example, /path/to/class-my-class.php should contain the declaration of
 * class My_Class.
 *
 * Intentionally does not check if a "register" method exists in the class
 * or if the class has been properly included. Doing so would hide ACTUAL errors
 * due to formatting mistakes that should, indeed, be noticed and fixed!
 *
 * @link https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/#naming-conventions
 *
 * @param string $file The full class filename.
 */
function register_class_from_file( string $file ) {

	$class_name = str_replace(
		[ 'class-', '-' ],
		[ '', '_' ],
		basename( $file, '.php' )
	);
	$class_name = __NAMESPACE__ . '\\' . ucwords( $class_name, '_' );

	if ( ! class_exists( $class_name ) ) {
		require_once $file;
		$class_name::register();
	}
}

/* Register Public Functionality */
foreach ( glob( PLUGIN_PATH . '/src/public/class-*.php' ) as $file ) {
	register_class_from_file( $file );
}

if ( is_admin() ) {
	/* Register Admin-Only Functionality */
	foreach ( glob( PLUGIN_PATH . '/src/admin/class-*.php' ) as $file ) {
		register_class_from_file( $file );
	}
} else {
	/* Register Frontend-Only Functionality */
	foreach ( glob( PLUGIN_PATH . '/src/public/frontend/class-*.php' ) as $file ) {
		register_class_from_file( $file );
	}
}
```

## bundle.sh

Place this bash script in the root folder of your plugin. I typically name it `bundle.sh`.

It zips your necessary plugin files into an installable plugin package with the current version appended.

```bash
#!/bin/bash

PLUGIN_SLUG=$( basename `pwd` )

VERSION=$( grep -Eio 'Version:\s*[0-9\.]+' "${PLUGIN_SLUG}.php" | grep -Eo '[0-9\.]+' )

cd ..
zip -rT9X "${PLUGIN_SLUG}-${VERSION}.zip" "${PLUGIN_SLUG}" --exclude '*/.git*' '*/.DS_Store' '*.zip' '*.log' '*.sh'
```

### Separate exclude.lst

Instead of listing the exact exclude globs in the `bundle.sh` script, you can instead specify a separate file. This is helpful for long lists, generic implementations, or shared excludes such as would be used by pipeline logic.

The `zip` command becomes this:

```bash
zip -rT9X "${PLUGIN_SLUG}-${VERSION}.zip" "${PLUGIN_SLUG}" --exclude @"${PLUGIN_SLUG}"/exclude.lst
```

And the `exclude.lst` file contains something like this:

```
completionist/*.zip
completionist/*.log
completionist/*.map
completionist/*.sh
completionist/*.DS_Store
completionist/.git*
completionist/action.yml
completionist/exclude.lst
completionist/composer.json
completionist/composer.lock
completionist/node_modules/*
completionist/package-lock.json
completionist/package.json
completionist/assets/styles/scss/*
completionist/src/components/*
completionist/src/*.js
completionist/src/*.jsx
```

Notice that the paths are the absolute zip package paths, which begin with the plugin directory's name (ie. plugin slug).
