---
title: WooCommerce
parent: Useful Commands
nav_order: 1
---

# Useful Commands: WooCommerce

## Apply Coupon Code via URL

WooCommerce hook to apply coupon code via the URL. Combine with the `?add-to-cart={product_id}` parameter to share a product with a coupon offer.

```php
/**
 * Apply WooCommerce coupon codes via GET param.
 */
add_action( 'woocommerce_after_calculate_totals', function() {

	if ( isset( $_GET['coupon_code'] ) && ! empty( $_GET['coupon_code'] ) ) {

		$coupon_code = filter_var(
			wp_unslash( $_GET['coupon_code'] ),
			FILTER_SANITIZE_STRING,
			FILTER_FLAG_STRIP_LOW | FILTER_FLAG_STRIP_HIGH
		);

		if (
			! empty( $coupon_code )
			&& class_exists( 'WooCommerce' )
			&& function_exists( 'WC' )
			&& ! WC()->cart->has_discount( $coupon_code )
			&& WC()->cart->get_total( 'edit' ) > 0
		) {
			WC()->cart->apply_coupon( $coupon_code );
		}
	}
});
```

## Ensure Disabled Checkout when Processing Order

WooCommerce does not always properly disable the Place Order button when checkout form is being submitted. This can lead to duplicate orders because the Place Order button is still clickable. The checkout loader spinner also gets whack, so just disable it and fade the checkout instead.

```html
<script type="text/javascript">
	jQuery(function() {

		jQuery( 'form.checkout' ).on( 'checkout_place_order', function() {
			jQuery( 'button#place_order' )
				.css( 'pointer-events', 'none' )
				.prop( 'disabled', true )
				.html( '<i class="fas fa-sync-alt fa-spin"></i> Processing...' );
		});

		jQuery( document.body ).on( 'checkout_error', function() {
			jQuery( 'button#place_order' )
				.css( 'pointer-events', 'auto' )
				.prop( 'disabled', false )
				.html( 'Place Order' );
		});

	});
</script>

<style>
	div.blockOverlay {
		display: none !important;
	}

	form.checkout.processing {
		opacity: 0.6 !important;
	}

	form.checkout.processing button#place_order {
		pointer-events: none !important;
	}

	form.checkout button#place_order {
		pointer-events: auto;
	}
</style>
```

## Export Orders Containing ONLY Certain Line Items

Uses the WooCommerce Orders Export plugin.

```php
add_filter( 'woe_order_export_started', function( $order_id ) { 

	// Return ONLY orders that contain these products ONLY.
	$found_product_ids = [
		'product_11057' => false,
		'product_538' => false
	];

	$order = new WC_Order( $order_id );
	$order_items = $order->get_items();

	foreach ( $order->get_items() as $item_id => $item_values ) {
		$product_id = $item_values->get_product_id();
		if ( isset( $found_product_ids[ "product_{$product_id}" ] ) ) {
			$found_product_ids[ "product_{$product_id}" ] = true;
		} elseif ( false === wc_pb_is_bundled_order_item( $item_values, $order ) ) {
			// Abort. An undesired product is in this order.
			return false;
		}
	}

	// Verify we found all products we wanted.
	foreach ( $found_product_ids as $is_in_order ) {
		if ( false === $is_in_order ) {
			// Abort. Not all required products are in this order.
			return false;
		}
	}

	// Order passed successfully and should be in the report.
	return $order_id;
});
```

## Dynamically Add Promotional Product to Cart

This code was written for a client that wanted to add a freebie product to the customer's cart as long as their cart contained a qualifying product. A higher-tier freebie would be added instead if the customer's cart contained a qualifying product AND exceeded a $100 subtotal.

```php
// Add promotional product to cart for qualifying orders.
// Compatible with WooCommerce's cart AJAX loading.
add_action( 'woocommerce_before_cart', 'ptc_modify_cart_items', 999, 1 );
function ptc_modify_cart_items() {

	// Define qualifying cart item products IDs.
	$qualifying_product_ids = array(
		123456,
		789012,
		345678,
		901234,
		567890,
	);

	// Define possible reward offers to add/remove from cart.
	$standard_gift_product_id = 100234;
	$premium_gift_product_id  = 560078;

	// Check the cart to see if qualified for promotional offer.

	$cart = WC()->cart;

	$found_qualifying_product_in_cart = false;

	$standard_gift_cart_item_key = '';
	$premium_gift_cart_item_key = '';

	foreach ( $cart->get_cart() as $cart_item_key => $cart_item ) {
		if (
			in_array( $cart_item['product_id'], $qualifying_product_ids ) ||
			in_array( $cart_item['variation_id'], $qualifying_product_ids )
		) {
			// Found qualifying product ID.
			$found_qualifying_product_in_cart = true;
		} elseif (
			$standard_gift_product_id == $cart_item['product_id'] ||
			$standard_gift_product_id == $cart_item['variation_id']
		) {
			// Found standard gift already in cart.
			$standard_gift_cart_item_key = $cart_item_key;
		} elseif (
			$premium_gift_product_id == $cart_item['product_id'] ||
			$premium_gift_product_id == $cart_item['variation_id']
		) {
			// Found premium gift already in cart.
			$premium_gift_cart_item_key = $cart_item_key;
		}
	}

	// Always remove reward cart items until qualified.

	if ( ! empty( $standard_gift_cart_item_key ) ) {
		$cart->remove_cart_item( $standard_gift_cart_item_key );
	}

	if ( ! empty( $premium_gift_cart_item_key ) ) {
		$cart->remove_cart_item( $premium_gift_cart_item_key );
	}

	// Add applicable reward if cart qualifies.
	if ( true === $found_qualifying_product_in_cart ) {
		if ( $cart->get_subtotal() >= 100 ) {
			$cart->add_to_cart( $premium_gift_product_id );
		} else {
			$cart->add_to_cart( $standard_gift_product_id );
		}
	}
}
```

