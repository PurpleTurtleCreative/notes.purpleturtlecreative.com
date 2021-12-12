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
 * The full file path to this plugin's main file.
 *
 * @since 1.0.0
 */
define( __NAMESPACE__ . '\PLUGIN_FILE', __FILE__ );

/**
 * The full file path to this plugin's directory ending with a slash.
 *
 * @since 1.0.0
 */
define( __NAMESPACE__ . '\PLUGIN_PATH', plugin_dir_path( __FILE__ ) );

/**
 * This plugin's current version.
 *
 * @since 1.0.0
 */
define( __NAMESPACE__ . '\PLUGIN_VERSION', get_file_data( __FILE__, [ 'Version' => 'Version' ], 'plugin')['Version'] ?? '0.0.0' );

/**
 * This plugin's basename.
 *
 * @since 1.0.0
 */
define( __NAMESPACE__ . '\PLUGIN_BASENAME', plugin_basename( __FILE__ ) );

/**
 * The full url to this plugin's directory, ending with a slash.
 *
 * @since 1.0.0
 */
define( __NAMESPACE__ . '\PLUGIN_URL', plugins_url( '/', __FILE__ ) );

/* CODE REGISTRATION */

// @TODO: Hook custom code.
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

