---
title: Gutenberg Colors
parent: WordPress
nav_order: 1
---

# Gutenberg Editor to Use SCSS Colors

Automatically load SCSS colors list names and values into WordPress Gutenberg editor.

```php
/**
 * Configure the Gutenberg Editor.
 */
add_action( 'after_setup_theme', function() {

	// Disable theme overrides from being applied.
	add_theme_support( 'disable-custom-gradients' );
	add_theme_support( 'disable-custom-colors' );
	add_theme_support( 'disable-custom-font-sizes' );

	// Define color palette.
	add_theme_support( 'editor-color-palette', get_custom_colors() );

}, 10 );

/**
 * Get color values defined in _colors.scss
 *
 * @see /assets/styles/sass/abstracts/variables/_colors.scss
 */
function get_custom_colors() {

	$colors = [];

	try {

		$file_contents = file_get_contents( get_template_directory() . '/assets/styles/sass/abstracts/variables/_colors.scss' );

		$colors_map = [];
		if ( preg_match( '/\$colors: \([^;]*\);/', $file_contents, $colors_map ) ) {

			if ( isset( $colors_map[0] ) ) {
				$colors_map = $colors_map[0];
			} else {
				throw new \Exception( 'Could not get $colors list.' );
			}

			$color_matches = [];
			if ( preg_match_all( '/(?P<slug>[a-z\-]+)\:\s*(?P<color>#[0-9abcdef]{3,6})/i', $colors_map, $color_matches ) ) {

				if (
					isset( $color_matches['slug'] )
					&& $color_matches['slug']
					&& isset( $color_matches['color'] )
					&& $color_matches['color']
					&& count( $color_matches['slug'] ) === count( $color_matches['color'] )
				) {

					foreach ( $color_matches['slug'] as $i => $slug ) {
						$colors[] = [
							'name' => ucwords( str_replace( '-', ' ', $slug ) ),
							'slug' => $slug,
							'color' => $color_matches['color'][ $i ],
						];
					}
				} else {
					throw new \Exception( 'Something went wrong with $color_matches.' );
				}
			} else {
				throw new \Exception( 'Could not get $color_matches.' );
			}
		} else {
			throw new \Exception( 'Could not match $colors list variable.' );
		}
	} catch ( \Exception $e ) {
		error_log( 'Failed to get_custom_colors. ' . $e->getMessage() );
		return [];
	}

	return $colors;
}
```

