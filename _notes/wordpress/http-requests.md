---
title: HTTP Requests
parent: WordPress
nav_order: 1
---

# HTTP Requests in WordPress

The simplest way to send HTTP requests in WordPress using PHP.

Here's helpful references to WordPress's documentation:

- [`add_query_arg()`](https://developer.wordpress.org/reference/functions/add_query_arg/) – Add query parameters to a URL.
- [`wp_remote_request()`](https://developer.wordpress.org/reference/functions/wp_remote_request/) – Send an HTTP request and retrieve its response.
- [`wp_remote_retrieve_response_code()`](https://developer.wordpress.org/reference/functions/wp_remote_retrieve_response_code/) – Retrieve only the response code from the raw response.
- [`wp_remote_retrieve_body()`](https://developer.wordpress.org/reference/functions/wp_remote_retrieve_body/) – Retrieve only the body from the raw response.

## POST

***Note that headers in WordPress's function are defined as key-value pairs while PHP's cURL headers are defined as an array of strings. See [HTTP Requests in PHP](https://notes.purpleturtlecreative.com/php/http-requests/) for more information.*

```php
// Prepare request location.
$request_url = add_query_arg(
	[
		'api_key' => \PTC_API_KEY,
		'api_secret' => \PTC_API_SECRET,
	],
	$this->api_url . '/api/2/contacts'
);

// Send the request.
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

// @DEBUG Inspect the response.
error_log( print_r( $response, true ) );

// Validate the response.
$http_code = (int) wp_remote_retrieve_response_code( $response );
if ( 200 !== $http_code ) {
  return [];
}

// Get the response body.
$body = wp_remote_retrieve_body( $response );
if ( empty( $body ) ) {
  return null;
}

// Decode the JSON response.
$response = json_decode( $body, true );// true = return associative array.
if ( ! is_array( $response ) ) {
  return null;
}

// Return the desired data, if available.
return $response['contacts'] ?? [];
```

