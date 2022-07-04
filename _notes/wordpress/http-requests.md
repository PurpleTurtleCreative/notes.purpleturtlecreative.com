---
title: HTTP Requests in WordPress
parent: WordPress
nav_order: 1
---

# HTTP Requests in WordPress

The simplest way to send HTTP requests in WordPress using PHP.

Here's helpful references to WordPress's documentation:

- [`add_query_arg()`](https://developer.wordpress.org/reference/functions/add_query_arg/) – Add query parameters to a URL.
- [`wp_remote_request()`](https://developer.wordpress.org/reference/functions/wp_remote_request/) – Send an HTTP request and retrieve its response.

## POST

***Note that headers in WordPress's function are defined as key-value pairs while PHP's cURL headers are defined as an array of strings. See [HTTP Requests in PHP](https://notes.purpleturtlecreative.com/php/http-requests/) for more information.*

```php
// Prepare request location.
$request_url = add_query_arg(
	[
		'api_key' => \PTC_API_KEY,
		'api_secret' => \PTC_API_SECRET,
	],
	'https://purpleturtlecreative.com/'
);

// Send the event to be recorded.
$response = wp_remote_request(
	$request_url,
	[
		'method' => 'POST',
		'headers' => [
			'Content-Type' => 'application/json',
		],
		'body' => json_encode(
			[
				'client_id' => \PTC_CLIENT_ID,
				'is_boolean' => true,
				'data' => $data,
			]
		),
	]
);

// Inspect the response.
error_log( print_r( $response, true ) );
```

