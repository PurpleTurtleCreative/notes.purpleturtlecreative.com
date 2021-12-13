---
title: Custom REST API
parent: WordPress
nav_order: 1
---

# Custom REST API with WordPress

Define the main REST API namespace in your plugin main file:

```php
namespace PTC_Plugin;

defined( 'ABSPATH' ) || die();

/**
 * The namespace for all v1 REST API routes registered by this plugin.
 *
 * @var string REST_API_NAMESPACE_V1
 *
 * @since 1.0.0
 */
define( __NAMESPACE__ . '\REST_API_NAMESPACE_V1', 'ptc-plugin/v1' );

// Hook custom code.
require_once PLUGIN_PATH . 'src/public/class-rest-server.php';
REST_Server::register();
```

Create a class file to register all REST API routes:

```php
<?php
/**
 * Registers REST API routes.
 *
 * @since 1.0.0
 */

namespace PTC_Plugin;

defined( 'ABSPATH' ) || die();

require_once PLUGIN_PATH . 'src/includes/rest-api/class-resource.php';
require_once PLUGIN_PATH . 'src/includes/rest-api/class-another-resource.php';

if ( ! class_exists( __NAMESPACE__ . '\REST_Server' ) ) {
	class REST_Server {

		public static function register() {
			add_action( 'rest_api_init', [ __CLASS__, 'register_routes' ] );
		}

		public static function register_routes() {
			REST_API\Resource::register_routes();
			REST_API\Another_Resource::register_routes();
		}//end register_routes()
	}
}
```

Create a class file for each route resource. Here's an example:

```php
<?php
/**
 * Registers resource REST API routes.
 *
 * @since 1.1.0
 */

namespace PTC_Plugin\REST_API;

defined( 'ABSPATH' ) || die();

if ( ! class_exists( __NAMESPACE__ . '\Resource' ) ) {
	class Resource {

		public static function register_routes() {

			register_rest_route(
				\PTC_Plugin\REST_API_NAMESPACE_V1,
				'/resource',
				[
					[
						'methods' => 'POST',
						'callback' => [ __CLASS__, 'post_new_resource' ],
						'permission_callback' => function() {
							if ( ! is_user_logged_in() ) {
								return new \WP_Error( 401, 'You must be logged in.' );
							}
							return true;
						},
						'args' => [
							'ptc_nonce' => [
								'type' => 'string',
								'required' => true,
								'validate_callback' => function( $value, $request, $param ) {
									$action = "ptc-plugin_{$request['resource_id']}";
									return in_array( wp_verify_nonce( $value, $action ), [ 1, 2 ], true );
								},
							],
							'resource_id' => [
								'type' => 'integer',
								'required' => true,
							],
						],
					], // END readable endpoint.
				]
			);//register_rest_route :: /resource
		}//end __construct()

		public static function post_new_resource( \WP_REST_Request $request ) {
			$response = [
				'status' => 'success',
				'data' => $request['resource_id']
			];
			return new \WP_REST_Response( $response, 200 );
		}
	}//end class
}
```

