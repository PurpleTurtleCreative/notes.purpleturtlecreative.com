---
title: Simple Toolbox Setup
parent: PHP
nav_order: 1
---

# Simple Toolbox Setup

To set up a simple PHP script website framework with Docker, use the following configurations...

*docker-compose.yml*

```yaml
version: '3.2'

services:

  php:
    image: php:8.0-apache
    ports:
      - "8000:80"
    working_dir: /var/www/html
    volumes:
      - ./:/var/www/html
      - ./php.ini:/usr/local/etc/php/conf.d/php.ini:ro
```

*php.ini*

```php
;;
; Errors
;;

error_reporting = 999999999
display_errors = false
log_errors = true
error_log = ./debug.log
html_errors = true
report_memleaks = true
log_errors_max_len = 0

;;
; Memory
;;

memory_limit = 128M
max_execution_time = 300

;;
; Time
;;

date.timezone = America/New_York
```

*src/header.php*

```php
<?php
/**
 * Global configurations and header display for PHP scripts.
 */

// Document paths.
define( 'PATH', $_SERVER['DOCUMENT_ROOT'] . '/' );

/* DEFINE OTHER GLOBALS HERE */

// Basic styles.
?>
<style>
html, body {
	margin: 0.5em;
	padding: 0;
	font-family: sans-serif;
}

header {
	background: #3c56f5;
	padding: 1em 0.5em;
	margin: -1em -1em 2em;
}

header a {
	margin: 0 1em;
	color: white;
	text-decoration: none;
}

header a.active {
	opacity: 0.5;
	pointer-events: none;
	cursor: default;
}

form {
	display: inline-block;
	background: #eee;
	padding: 1em 2em;
	border: 1px solid #ccc;
}

form label {
	display: block;
	margin: 1em 0;
}

form select,
form input[type=text] {
	font-size: 0.9em;
	padding: 0.3em;
	min-width: 300px;
}

form input[type=submit] {
	background: #3c56f5;
	color: white;
	border: none;
	font-size: 1em;
	padding: 0.7em 1em;
	margin: 1em 0;
	border-radius: 3px;
	cursor: pointer;
	letter-spacing: 0.05em;
}

form input[type=submit]:hover {
	background: #1123b0;
}
</style>
<?php

// Display script navigation header.

$scripts = glob( PATH . 'src/*.php' );
array_unshift( $scripts, PATH . 'index.php' );// Prepend homepage.

echo '<header>';
foreach ( $scripts as $file ) {
	if ( $file === __FILE__ ) {
		continue;// Don't include this header.php file.
	}
	$href = str_replace( $_SERVER['DOCUMENT_ROOT'], '', $file );
	$title = ucwords( str_replace( '-', ' ', basename( $file, '.php' ) ) );
	$class = ( $file === $_SERVER['SCRIPT_FILENAME'] ) ? 'active' : '';
	echo "<a href='{$href}' class='{$class}'>{$title}</a>";
}
echo '</header>';
```

*src/example-script.php*

```php
<?php
/**
 * Example script.
 */

// Include global settings and header navigation bar. 
require_once 'header.php';

// Frontend form.
?>
<form action="" method="GET" autocomplete="off">
	<input type="submit" name="form_submit" value="Run" />
</form>
<?php

###############################################
### -------! Form Processing Below !------- ###
###############################################

if ( empty( $_GET['form_submit'] ) ) {
	exit;
}

echo 'Hello, World!';

exit;
```

