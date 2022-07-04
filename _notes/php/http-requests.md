---
title: HTTP Requests in PHP
parent: PHP
nav_order: 1
---

# HTTP Requests in PHP

Use the cURL functions in PHP to make HTTP requests and observe the response.

Here's some helpful references from PHP's documentation:

- **cURL Options:** [https://www.php.net/manual/en/function.curl-setopt.php](https://www.php.net/manual/en/function.curl-setopt.php)
- **cURL Info Options:** [https://www.php.net/manual/en/function.curl-getinfo.php](https://www.php.net/manual/en/function.curl-getinfo.php)
- **cURL Multiple (Asynchronous) Requests:** [https://www.php.net/manual/en/function.curl-multi-init.php](https://www.php.net/manual/en/function.curl-multi-init.php)

## POST

```php
// Initialize a new session handle.
$ch = curl_init();
curl_setopt_array(
	$ch,
	[
		CURLOPT_RETURNTRANSFER => true,
		CURLOPT_POST => true,
		CURLOPT_URL => 'https://functionschallenge.digitalocean.com/api/sammy',
		CURLOPT_HTTPHEADER => [
			'Accept: application/json',
			'Content-Type: application/json'
		],
		CURLOPT_POSTFIELDS => json_encode( $args ),
	]
);

// Inspect the response.
error_log( print_r( json_decode( curl_exec( $ch ) ), true ) );
error_log( print_r( curl_getinfo( $ch ), true ) );

// Close the session and free resources.
curl_close( $ch );
```

