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

